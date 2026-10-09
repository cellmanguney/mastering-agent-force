# Traps

Things that cost time while this course was being developed. Each has the date it was seen. Check whether it still applies before repeating it to a learner.

## Stale material

Most Agentforce tutorials online describe an older product.

| You read | It means now | Since |
|---|---|---|
| "Topics" | Subagents. Same thing, renamed | April 2026 |
| "Data Cloud" | Data 360 | 2025 |
| Setup → Agentforce Agents → New Agent, builder with separate Topics / Actions tabs | Legacy builder. New agents are created only in Agentforce Builder inside Agentforce Studio | Mid-July 2026 |
| "Agentforce Sales", "Agentforce Commerce" | Sales Cloud, Commerce Cloud. "Agentforce" names the agent platform | 2026 |
| Instructions written as prose only, "Topics, Instructions, Actions" | Agent Script: logic and prompts in one file | GA early 2026 |

## Org and sign-up

- A Developer Edition is not a sandbox. It logs in as **Production**. (2026-10-09)
- A Developer Edition created before Agentforce was included may lack Agentforce Studio. Sign up for a new one. (2026-10-09)
- A new Developer Edition can arrive with sample agents and over a hundred sample Flows. They are not the learner's work and the course ignores them. (2026-10-09)

## Builder

- **Preview runs the saved version.** An edit that isn't saved is not tested. Check "Last saved" in the top bar before every test. (2026-10-09)
- **Variables last for the session.** Use the refresh icon in Preview for a clean session before each test. (2026-10-09)
- **The API name field is labeled "Developer Name"** in the "Name your agent" dialog. In script it is `developer_name`. (2026-10-09)
- **Undo may not work after a save.** Cmd+Z did nothing in Script view after saving; retype instead. Cause not confirmed. (2026-10-09)
- **The Flows list page has no type picker.** "Autolaunched Flow (No Trigger)" appears only after clicking New Flow, in the New Automation dialog. (2026-10-09)

## Script

- **Indentation is significant.** A gate or `set` at the wrong depth attaches to the wrong thing or fails validation.
- **Unset is `None`, not `""`.** A mutable string without a default has no value. Test with `is None`.
- **Flow variable names must match the action's inputs and outputs exactly**, and `flow://` takes the Flow's API name, exact in capitals and underscores.
- **A prompt for the post-action state is read right after the action too.** Instructions are resolved again in the same turn once a variable changes. "A return already exists" will be said about the return that was just created. (2026-10-09)
- **An ungated `run` in `reasoning` fires on every parse**, not once per turn.

## Seen but not explained

Don't present guesses about these as facts. If a learner sees the same, record it.

- After removing the router's `model_config` lines, the placeholder-target warnings became 2 errors: "Validation failed for action(s) '...' due to invalid attribute value for 'source'. Please specify a source when using an OOTB target." The edit was correct. The errors went away once the targets pointed at real Flows. (2026-10-09)
- A URL returned by a Flow showed as `URL_Redacted` in the agent's reply. The Summary attributed it to trust policies. How to allow a legitimate link has not been worked out. (2026-10-09)
- In one run the Summary showed three consecutive reasoning passes in the same subagent before the action ran. Not investigated in the Trace. (2026-10-09)

## Documentation that disagrees with itself

- Whether a new utterance restarts at the router or resumes in the last subagent. Observed with default settings: restarts at the router. (2026-10-09)
- Whether agent-to-agent orchestration (connected subagents) is Beta or generally available. Check current release notes.
