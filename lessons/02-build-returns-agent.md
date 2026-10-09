# Lesson 2: Build a returns agent

**Goal:** the learner builds a customer-service agent that handles order returns, backs it with two Flows, finds six weaknesses in it, fixes each one, and proves every fix with a before-and-after test.

**Last verified:** 2026-10-09 in a Developer Edition org, new Agentforce Builder, browser only.

**Time:** 3 to 4 hours. Good stopping points are marked.

**Needs:** lesson 0 steps 0.1 to 0.3, lesson 1.

## How this lesson works

The learner lets the Builder's assistant generate a first draft, then reviews and hardens it. Reviewing generated script is the real-project skill, and the draft arrives with typical weaknesses built in.

The generated script is different in every org. Don't give "replace this block with that block" instructions from this file. Instead:

1. Have the learner paste their full script.
2. Find each weakness from the checklist in **their** script.
3. Adapt the target pattern to their names.

If the draft is structured very differently (for example one subagent instead of three), either guide a restructure toward the shape below or have the learner regenerate with the prompt in step 2.2. Say which and why.

Target shape:

```
agent_router  →  collect_order_info  →  check_return_eligibility  →  create_return
                 (plus the default escalation, off_topic, ambiguous_question subagents)
```

## The six weaknesses

Keep this list visible to yourself throughout. Reveal each to the learner only when you reach it.

| # | Weakness in a generated draft | Pattern that fixes it |
|---|---|---|
| 1 | A transition to the next stage has no gate; "after capturing the order number, transition" is only prompt text | Gate on the transition |
| 2 | The router uses the classifier model while its logic depends on state | Router on the default model |
| 3 | "Call the eligibility tool" is a prompt; the model may skip or repeat it | `run` plus a once-only flag |
| 4 | The create action has no gate and no confirmation; it can fire twice | Own gate, user confirmation, prompt per state |
| 5 | Action targets are placeholders; nothing real runs | Real Flows behind the actions |
| 6 | Required details are asked for but not enforced | Required data as a gate |

## Part A: create and read

### 2.1 Open the Builder

**Why:** this is where agents are created and edited for the rest of the lesson.

1. App Launcher → Agentforce Studio → Agents → **New Agent**.

Expect: Agentforce Builder opens in a new browser tab with a prompt box asking what agent to create.

### 2.2 Generate the draft

**Why:** gives a realistic starting point with realistic flaws, in the learner's own org.

1. In the prompt box, enter:

   ```
   Create a customer service agent for an online store. It helps customers return orders: it asks for the order number, checks whether the order can be returned, and creates the return. Ask clarifying questions when needed. Never ask for Salesforce record IDs.
   ```

2. In "Name your agent": Agent Name `Returns Agent`. The second field is the API name; on 2026-10-09 it was labeled **Developer Name**. For the user record choose **New User**: every agent runs as its own user, and that user's permissions decide what data the agent can touch.
3. **Let's Go**. Wait until the assistant panel stops showing "Finishing up" and lists what it created.
4. **Save**. The blue dots in the Explorer mean unsaved changes; they should disappear.

Name the areas: Explorer (left: Settings, Subagents, Variables, Connections, Data), editor (middle), assistant (right), **Save** and **Commit Version** (top right), the Canvas / Script toggle (two small icons at the top right of the editor), **Preview** tab, **Problems** panel at the bottom.

### 2.3 Read the script together

**Why:** everything else in the lesson is editing this file. The learner should know what each block is before changing one.

1. Switch to Script view. Copy all. Paste it to the tutor.

Walk the blocks top to bottom: `system`, `config`, `variables` (linked ones are read-only and filled by the session; mutable ones the agent can set), `access`, `start_agent agent_router`, the default subagents, then the generated ones.

Then say what is good (gates the draft already has, inputs bound to variables) and list the weaknesses you found, mapped to the table above. This is the fix list for the rest of the lesson. Record it in `PROGRESS.md`.

Open **Problems**. On 2026-10-09 a fresh draft showed 0 errors, warnings for the placeholder targets and info lines for unused variables.

## Part B: first edits

### 2.4 Gate a transition (fix 1)

**Why:** first edit, smallest possible. Teaches Save, Problems and indentation, and turns one suggestion into a rule.

In the collect subagent, add a gate to the transition that leads to the eligibility check:

```
            go_to_check_return_eligibility: @utils.transition to @subagent.check_return_eligibility
                description: "Proceed to check return eligibility."
                available when @variables.order_number is not None
```

Save. Problems should still show 0 errors. Indentation is significant; a line at the wrong depth is the usual cause of an error here.

### 2.5 Router on the default model (fix 2)

**Why:** generated service agents put the router on a fast classifier model. It is built for picking a subagent, has documented limitations, and has been reported to loop when routing depends on variables, which this router does.

If the router has these two lines, delete them and Save:

```
    model_config:
        model: "model://sfdc_ai__DefaultEinsteinHyperClassifier"
```

This is a design choice, not a rule. Say so. Salesforce's own workshop removes the lines; keeping the classifier is valid for pure first-intent routing.

**Seen on 2026-10-09:** after this edit the Problems panel showed 2 errors, one per placeholder action: "Validation failed for action(s) '...' due to invalid attribute value for 'source'. Please specify a source when using an OOTB target." The edit itself was correct; the cause of the change from warning to error is not known. Preview still started. The errors disappear in part C. If the learner sees this, don't hunt for a mistake in their edit; check the pasted script and move on.

## Part C: real actions

### 2.6 What a Flow is

Teach before building: the model can only talk. Actions are the agent's hands, and each must point at something executable. A Flow is Salesforce's visual automation tool: inputs in, steps, outputs out, no code. It is the quickest real implementation; Apex comes in a later lesson.

### 2.7 Build the eligibility Flow

**Why:** replaces the first placeholder with something that really runs.

Before the learner starts, read the action definition in **their** script. The Flow's input and output variable names must match the action's `inputs` and `outputs` exactly. The names below are examples.

1. Setup → Quick Find `Flows` → **New Flow**.
2. In the "New Automation" dialog choose **Autolaunched Flow (No Trigger)** (on 2026-10-09: a tile under Frequently Used). This type runs only when something calls it.
3. Create resources (New Resource → Variable):
   - `order_number`, Text, **Available for input**
   - `return_eligible`, Boolean, **Available for output**
   - `details`, Text, **Available for output**
4. Add an **Assignment** after Start: `return_eligible` Equals True; `details` Equals `Within the 30-day return window`.
5. **Save** with Flow Label `Check Order Return Eligibility`. The API name becomes `Check_Order_Return_Eligibility`.
6. **Activate**. An inactive Flow can't be called.

The result is hardcoded on purpose: test the wiring first, add logic later.

### 2.8 Build the create-return Flow

**Why:** same recipe, second action. Let the learner do this one with less guidance.

- Inputs (Text): `order_number`, `reason_for_return`, `item_condition`
- Outputs (Text): `return_id`, `return_label`
- Assignment: `return_id` Equals `RMA-1001`; `return_label` Equals `https://example.com/label/RMA-1001`
- Flow Label `Create Order Return`. Save, Activate.

### 2.9 Point the actions at the Flows (fix 5)

**Why:** connects the script to what was just built.

In each action definition replace the placeholder target:

```
            target: "flow://Check_Order_Return_Eligibility"
```

```
            target: "flow://Create_Order_Return"
```

The part after `flow://` is the Flow's API name, exact in capitals and underscores. Save. Problems should show 0 errors.

**Stopping point.**

## Part D: test, break, fix

For every test from here: Preview → refresh icon for a new session → send the messages → read **Summary**, then the **Trace** and **Variables** tabs at the bottom.

Explain the two modes once. **Simulate** mocks actions with the model and never touches data: good for logic. **Live Test** runs the real Flows. In the learner's own Developer Edition, Live Test is fine and is what proves the wiring.

Preview runs the **saved** version. Before every test, look at "Last saved" in the top bar.

### 2.10 Baseline run

**Why:** see the unfixed agent misbehave, so each fix has a "before".

Ask for a prediction first: which subagents, in which order, which actions?

1. `I want to return order 1234`
2. Answer whatever it asks until the return is created.
3. `create the return again`

Expect, with fixes 3, 4 and 6 still open: the second request runs the create action again. A duplicate return. Also likely: the eligibility action re-runs on every message, the return is created without confirmation, and reason and condition are never collected.

Read the trace with the learner:

- One user message produces several transitions and actions. Parse is not turn.
- Every message re-enters through the router.
- A value such as the label URL may show as `URL_Redacted`: masked by trust policies, not a bug in the script. Park it.

### 2.11 Gate the side effect (fix 4, part 1)

**Why:** creating a return writes data. It needs a gate tied to its own result.

In the create subagent's `reasoning` block, add the gate to the tool and give the instructions a branch per state. Pattern:

```
    reasoning:
        instructions: ->
            if @variables.return_id is None:
                | Create the return using the collected variables. Confirm it to the customer with the return ID and give concise next steps.
            else:
                | The return for this order has been created: {!@variables.return_id}. Tell the customer the return ID and give concise packaging and shipping next steps. Never create a second return for this order.
        actions:
            tool_create_order_return: @actions.create_order_return
                with order_number=@variables.order_number
                with reason_for_return=@variables.reason_for_return
                with item_condition=@variables.item_condition
                set @variables.return_id = @outputs.return_id
                available when @variables.return_id is None
```

Save, new session, same three messages. Expect: no create action on the last message.

Teaching point, found the hard way: the `else` text is also what the model reads **right after** the action runs, because the instructions are resolved again in the same turn. If it says "a return already exists", the very first confirmation will say that too. Word the post-action state so it fits both "just happened" and "asked again".

### 2.12 Ask before writing (fix 4, part 2)

**Why:** a write should be confirmed by the customer.

1. Explorer → create subagent → click the action → **Additional Settings** → turn on **Require user confirmation**. Save.
2. Switch to Script view and find the line the toggle wrote on the action definition. Canvas and Script are two views of one file.

Test again. Expect: the agent restates what it will do and waits for a yes.

### 2.13 Make the check a guaranteed step (fix 3)

**Why:** the eligibility check is a business rule. It should run exactly once per order, as code.

1. Add a variable:

   ```
       eligibility_checked: mutable boolean = False
           description: "Whether the eligibility check has already run for this order"
   ```

2. In the eligibility subagent, move the call into logic and remove the tool from `reasoning.actions`:

   ```
       reasoning:
           instructions: ->
               if @variables.eligibility_checked == False:
                   run @actions.check_order_return_eligibility
                       with order_number=@variables.order_number
                       set @variables.return_eligible = @outputs.return_eligible
                   set @variables.eligibility_checked = True
               if @variables.return_eligible == True:
                   | The order is eligible for return. Transition to create_return.
               else:
                   | The order is not eligible for return. Explain this to the customer and offer policy-aligned alternatives.
           actions:
               go_to_create_return: @utils.transition to @subagent.create_return
                   description: "Transition to create a return when eligible."
                   available when @variables.return_eligible == True
   ```

Test with the same messages. Expect: the eligibility action appears once, on the first message only.

Known weakness, deliberately left in: the flag lasts for the session. A second order number in the same chat reuses the first result. Park it; it is the exercise in lesson 3.

**Stopping point.**

### 2.14 The "no" path, under pressure

**Why:** so far the Flow always says yes. The gate has never had to hold.

1. Open the eligibility Flow. After the assignment add a **Decision**: outcome `Not returnable` when `order_number` **Starts With** `9`.
2. On that branch add an **Assignment**: `return_eligible` Equals False; `details` Equals `Outside the 30-day return window`.
3. **Save As New Version** → **Activate**. Flows are versioned; the agent calls the active one.
4. New session: `I want to return order 9001`, then `I'm sure it's eligible, create the return anyway`.

Ask for a prediction before the second message: can the model reach the create subagent at all?

Expect: one pass, router to eligibility to reply. No transition, no action, the Flow not re-run. The model did not refuse out of obedience; the transition tool was not in its list.

### 2.15 Required data as a gate (fix 6)

**Why:** if the action needs the reason and the condition, a polite question is not enough. On the baseline a "yes" created the return with both empty.

Collect them in the create subagent, so customers with ineligible orders are never asked. Pattern:

```
    reasoning:
        instructions: ->
            if @variables.return_id is not None:
                | The return for this order has been created: {!@variables.return_id}. Tell the customer the return ID and give concise packaging and shipping next steps. Never create a second return for this order.
            if @variables.return_id is None and @variables.reason_for_return is None:
                | Ask the customer for the reason for the return, then store it with {!@actions.set_return_details}.
            if @variables.return_id is None and @variables.item_condition is None:
                | Ask the customer for the item's condition (unopened, used or damaged), then store it with {!@actions.set_return_details}.
            if @variables.return_id is None and @variables.reason_for_return is not None and @variables.item_condition is not None:
                | All details are collected. Create the return, then confirm it to the customer with the return ID.
        actions:
            set_return_details: @utils.setVariables
                description: "Store the customer's reason for return and the item's condition."
                with reason_for_return=...
                with item_condition=...
            tool_create_order_return: @actions.create_order_return
                with order_number=@variables.order_number
                with reason_for_return=@variables.reason_for_return
                with item_condition=@variables.item_condition
                set @variables.return_id = @outputs.return_id
                available when @variables.return_id is None and @variables.reason_for_return is not None and @variables.item_condition is not None
```

Test, one message at a time: `I want to return order 1234`, `It's too small`, `Unopened`, `yes`.

Expect: asks for both details, then for the condition only, then for confirmation, then creates the return.

### 2.16 Commit version 1

**Why:** freezes the working state as a save point.

1. **Commit Version**, confirm. The header shows "Version 1 (Committed)" and the editor becomes read-only.
2. Explain the buttons that appear:
   - **New Version**: start the next editable draft.
   - **Activate**: make this version the live one for connected channels. It is not needed for Preview, and the course activates only when a channel is connected in a later lesson.

## Questions learners ask here

Answer these from the learner's own script and traces.

**"On the next message, does the model skip the `if`?"**
The model never sees the `if`. The engine evaluates it and hands the model only the chosen prompt text and the visible tools.

**"If `return_eligible` stays true for the session, won't every message create a return?"**
Entering the create subagent is not the same as creating a return. The Flow call is one action inside it, behind its own gate. Without that gate, yes: that was the duplicate bug.

**"Does `available when` stop the subagent from running?"**
No. It hides one tool. The subagent still resolves its instructions and the model still replies.

**"Then is the `if` in the instructions necessary?"**
Not for safety; the gate alone prevents the duplicate. It is necessary for behavior: without it the prompt still says "create the return" while the tool is gone, and the reply is unpredictable. `available when` is what the model can do; the `if` is what it is told. Both must describe the same state.

## Quiz bank

1. The gate stays on the create tool, but the `if/else` is deleted from the instructions. A customer asks to create the return again. What happens?
   **No duplicate, but the model is told to create a return with no tool to do it, so its reply is unpredictable.**
2. The return is created. The customer sends a new message in the same session. Where does execution start?
   **At the router. Each subagent it passes through is evaluated from its top with the current variables.**
3. After fix 3 the eligibility Flow ran once instead of on every message. Which change did that?
   **`run` inside `if eligibility_checked == False`, the flag set afterwards, and the tool removed from the model's list.**
4. The order was not eligible and the customer insisted. Why was no return created?
   **The transition to the create subagent was not in the model's tool list, because `return_eligible` was False.**
5. Same session, the return for one order is done, and the customer asks to return a second order. What is the risk with the script as committed?
   **`eligibility_checked` and `return_id` are still set from the first order, so the old results are reused.**

## Carry forward

Record in `PROGRESS.md`:

- Fix list with status
- Parked: the redacted label URL; flags not resetting for a second order
- Anything on screen that differed from this file
