# Lesson 1: Agent Script concepts

**Goal:** the learner can read an Agent Script file and predict, roughly, what the engine does and what the language model sees.

**Last verified:** 2026-10-09, against the Agent Script documentation and the Salesforce Architects hybrid reasoning guide (see `reference/sources.md`), and against traces from a Developer Edition org.

**Time:** 45 minutes. No org work. Lesson 2 makes all of this concrete, so don't aim for full understanding here; aim for vocabulary and a model to test.

Teach in the four chunks below. After each, ask one check question before continuing.

## 1. What Agentforce is

Agentforce is Salesforce's platform for building agents. It runs on the core Salesforce platform (Flow, Apex, metadata, permissions), not on the B2C Commerce stack. Learners from a commerce background need that said early.

The parts:

- **Agent**: one Agent Script file. A declarative language holding configuration, business logic and prompts.
- **Subagent**: a scoped specialist with its own instructions and actions. Called "topics" until April 2026; only the name changed.
- **Action**: something executable the agent can call. A Flow, an Apex class, a prompt template.
- **Variables**: the session's state.
- **Runtime**: the script compiles to an Agent Graph. The Atlas Reasoning Engine walks it. Parts without a prompt run as code; parts with a prompt call the language model.
- **Around it**: Data 360 for grounding, the Trust Layer for masking and audit, channels such as chat, voice and Slack.

The skill the whole course trains: deciding where the boundary between code and the language model goes. It sets cost, latency and auditability. Default to code; use the model where judgment or natural language is needed.

Warn the learner about stale material, because most tutorials they will find are out of date:

- New agents can only be created in the new Agentforce Builder, inside Agentforce Studio (since mid-July 2026). Tutorials showing Setup → Agentforce Agents → New Agent are for the legacy builder.
- "Topics" means subagents. "Data Cloud" means Data 360.
- "Agentforce" is the agent platform. Sales Cloud, Service Cloud and Commerce Cloud carry their own names again.

Check question: which of Flow, subagent, variable is "something executable"?

## 2. Reading the syntax

Show this and walk it top to bottom. It is illustrative; the learner's real script in lesson 2 is the one that matters.

```
subagent returns:
    description: "Handles order returns"
    reasoning:
        instructions: ->
            if @variables.order_id is not None and @variables.checked == False:
                run @actions.check_eligibility
                    with order_id=@variables.order_id
                    set @variables.eligible=@outputs.eligible
                set @variables.checked = True
            if @variables.eligible == True:
                | Order {!@variables.order_id} is returnable. Confirm, then use {!@actions.do_return}.
            else:
                | Ask for the order number, or explain it can't be returned.
        actions:
            capture_order: @utils.setVariables
                description: "Store the order number the customer gives"
                with order_id = ...
            do_return: @actions.create_return
                with order_id=@variables.order_id
                set @variables.rma=@outputs.rma
                available when @variables.eligible == True and @variables.rma is None
```

A variable declared without a default has no value: test it with `is None`. A string declared with `= ""` is an empty string instead. Mixing the two up is a common reason a gate never opens.

| Syntax | Meaning |
|---|---|
| `->` | Logic mode: what follows can contain `if`, `run`, `set` |
| `\|` | A prompt line: text for the language model |
| `{!@variables.x}` | Insert a variable's value into prompt text |
| `run @actions.x` | Call an action now, as code |
| `with a=b` | Bind an action input |
| `with a = ...` | Let the language model fill this input from the conversation |
| `set @variables.x = @outputs.y` | Store an action output |
| `available when` | Gate: the tool exists for the model only while the condition is true |

Not shown: the `variables:` block, the `start_agent` router that every agent has, and action definitions with a `target` of `flow://`, `apex://` or `prompt://`.

There are two blocks named `actions`. `subagent.actions` defines what exists. `reasoning.actions` lists the tools the language model may choose. An action can be called by your logic with `run`, offered to the model as a tool, or both.

Check question: in the snippet, which lines can the language model influence?

## 3. How it executes

This is the core of the lesson. Teach it as a loop, then let lesson 2's traces prove it.

A mental model in code. It is a model, not Salesforce's implementation:

```js
// one session = one conversation
const vars = { /* the variables block, with defaults */ }
const history = []

function onUserMessage(text) {
  history.push(text)
  let current = "agent_router"                  // every message starts here
  while (true) {
    // ENGINE: run this subagent's script top-down (if, run, set).
    // Result: one prompt string + the list of visible tools.
    const { prompt, tools } = resolve(current, vars)

    // MODEL: sees only prompt + tools + history. Never the script.
    const out = llm(prompt, tools, history)

    if (out.type === "reply")      return out.text
    if (out.type === "transition") { current = out.target; continue }
    if (out.type === "tool")       { callTool(out, vars); continue }
  }
}
```

What to draw out:

- **Resolve, then reason.** The engine processes instructions top-down. Logic runs as code; `|` lines are collected into the prompt. The model starts only once the prompt is resolved. It never sees an `if`.
- **Parse is not turn.** Each pass of the loop is one parse. A parse happens on entering a subagent, after every tool result, and on every new user message. One user message can take five parses.
- **State is only variables and history.** Nothing remembers a position in the script. Variables live for the whole session.
- **`run` is a guarantee, a prompt is a suggestion.** "Always run X" in a prompt may or may not happen. `run` in logic always happens, on every parse where its `if` is true, so it usually needs a guard.
- **A gate removes the tool.** With `available when` false, the tool is not in the model's list. This is different from telling the model not to use it.
- **Transitions are one-way.** `@utils.transition to` hands over control and the target starts from its top. A direct `@subagent.x` reference as a tool is a call that returns.

Lifecycle blocks, for later lessons: `before_reasoning` and `after_reasoning` are code-only blocks around `reasoning`. `before_reasoning` runs on every parse, so it is not a constructor. `after_reasoning` is skipped if the subagent transitioned mid-run.

Where the documentation is unclear and what was observed:

- The flow-of-control page says every utterance returns to `start_agent`; the `reset_to_initial_node` setting reads as if the default were to resume. Observed on 2026-10-09 with default settings: every message re-entered at the router.
- Connected subagents (agent-to-agent) are tagged Beta in the developer docs while announcements call multi-agent orchestration generally available. Check release notes before relying on it.

Check question: a `run` sits in `reasoning` with no guard. The model makes two tool calls in one turn. How many times does the `run` fire? (Three: once per parse.)

## 4. Rules to carry into lesson 2

1. Every action with a side effect needs its own gate, tied to its own result. Don't rely on earlier stages to protect it.
2. The variable in a gate is set by code, never by the model, and defaults to closed.
3. A prompt and the visible tools must describe the same state. If the tool is hidden, the prompt must not ask for it.
4. The platform does not suppress an action after it runs. If the gate stays open, the model can call it again.
5. The model picks tools by name and description. Write descriptions as routing rules.
6. Shorter instructions behave more reliably.

## Quiz bank

Five questions. Adapt wording; keep the scenarios.

1. What does the language model receive from `reasoning.instructions`?
   **Only the resolved `|` text, after the logic has run.** Not the logic lines.
2. `before_reasoning` runs an action that writes a record. In one user turn the model makes two tool calls. How many records?
   **Three.** One per parse: at turn start and after each tool result.
3. `consult: @subagent.pricing` versus `go: @utils.transition to @subagent.pricing`?
   **The first returns to the caller; the second is one-way.**
4. A tool has `available when @variables.verified == True` and `verified` is False. What does the model experience?
   **The tool is absent from its tool list.**
5. The model calls `create_case` again on every parse. Most robust fix?
   **Set a variable in code after it runs so the gate turns false.** Prompt wording or a model-set flag stays probabilistic.

After the quiz, tell the learner honestly: this will feel abstract until they watch their own agent's trace in lesson 2. That is expected.
