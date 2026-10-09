# Mastering Agentforce

A hands-on course for Salesforce Agentforce, taught one-to-one by Claude.

You build everything yourself, in your own free Salesforce org, on your own machine. This repository holds the lesson plans and the tutor's rules. It ships no ready-made agent, no Flows and no shared org: if you didn't build it, you don't have it.

## What you need

- A computer where you can install software (macOS, Windows or Linux)
- An email address for a free Salesforce Developer Edition org
- Access to Claude with image input (you will send screenshots) and web search
- About 6 to 8 hours for lessons 0 to 3, in as many sittings as you like

No Salesforce experience is assumed. Experience with any programming language helps from lesson 2 onward.

Lesson 0 walks you through every install and the org sign-up. Don't prepare anything in advance.

## How to start

1. Clone this repository.

   ```
   git clone https://github.com/cellmanguney/mastering-agent-force.git
   cd mastering-agent-force
   ```

2. Open it with Claude.
   - **Claude Code**: start it in this folder. `CLAUDE.md` is loaded automatically.
   - **Claude app**: start a chat with this repository attached, or add the files to a project.

3. Paste the prompt from [`START.md`](START.md).

The tutor creates a local `PROGRESS.md` for you. It is git-ignored, stays on your machine, and is how a new chat picks up where the last one stopped.

### Which model

Use the most capable Claude model available to you. The course was developed in a session configured for the model ID `claude-fable-5-1`. Keep web search on: Agentforce changes monthly and the tutor is told to check current Salesforce documentation instead of trusting these files.

## Lessons

| # | Lesson | You end with | Status |
|---|---|---|---|
| 0 | [Prerequisites](lessons/00-prerequisites.md) | Your own org with Agentforce on, local tools installed | Org steps verified 2026-10-09. Local tool steps from docs, not yet run end to end |
| 1 | [Agent Script concepts](lessons/01-agent-script-concepts.md) | A mental model of how an agent executes | Verified 2026-10-09 |
| 2 | [Build a returns agent](lessons/02-build-returns-agent.md) | A working agent with two Flows, hardened and tested | Verified 2026-10-09 |
| 3 | [The agent as code](lessons/03-local-dx.md) | The same agent in VS Code, CLI and git | Draft |

Planned, not written yet: grounding with Data 360 and Prompt Builder, testing and observability, multi-agent and MCP, commerce agents, deploying between orgs, certification prep, pricing and sizing.

## How the course is built

- [`CLAUDE.md`](CLAUDE.md): the tutor's standing brief. Teaching style, step format, what to verify.
- [`lessons/`](lessons/): one plan per lesson, written for the tutor. Goal, steps, expected results, pitfalls, quiz.
- [`reference/`](reference/): traps we hit, and the official sources the lessons are based on.
- [`PROGRESS.template.md`](PROGRESS.template.md): the shape of your local progress file.

Lesson files are plans for the tutor, not handouts. Reading them alone will not teach you much; doing the steps will.

## How the course evolves

The course is shaped while it is being taken. Each lesson carries a "last verified" date. When your screen differs from a lesson, the tutor follows your screen, notes the difference in your `PROGRESS.md` under "Course feedback", and offers you the text for an issue or pull request. Please send it: the next learner gets a better lesson.
# mastering-agent-force
