# Lesson 3: The agent as code

**Status: draft.** This lesson has not been run end to end during course development. The commands come from the Agentforce DX documentation and the Salesforce CLI reference as read on 2026-10-09. Before each step, check the command with `--help` and against `reference/sources.md`, tell the learner it is unverified, and record what happened in "Course feedback".

**Goal:** the learner's returns agent lives in a local Salesforce DX project under git, and they can run the loop edit → validate → preview → publish without the browser.

**Needs:** lesson 0 complete (tools installed, CLI connected as `af-dev`), lesson 2 complete (a committed returns agent and two active Flows).

**Time:** estimate 2 hours.

## Concepts to teach first

- An agent is metadata, like everything else in Salesforce. Its Agent Script file lives in a metadata component called `AiAuthoringBundle` (the "authoring bundle"), as a file ending in `.agent`.
- A Salesforce DX project is a folder structure for that metadata. The CLI moves metadata between the folder and the org.
- git never deploys to Salesforce. Git versions the files; the CLI deploys them.
- Flows are metadata too. The agent's actions depend on them, so they belong in the project with the agent.

## Planned steps

### 3.1 Create the project

**Why:** a place for the agent's files, separate from this course repository.

```
sf project generate --name agentforce-lab
cd agentforce-lab
git init
code .
```

### 3.2 Pull the agent and its Flows down

**Why:** get what was built in the browser into files.

Planned commands. The exact member name of the bundle needs confirming against the org; `sf org list metadata --metadata-type AiAuthoringBundle` should show it.

```
sf project retrieve start --metadata AiAuthoringBundle
sf project retrieve start --metadata Flow:Check_Order_Return_Eligibility --metadata Flow:Create_Order_Return
```

Have the learner find the `.agent` file and compare it with Script view in the browser. Then the first commit:

```
git add .
git commit -m "Returns agent as built in lesson 2"
```

### 3.3 Validate and preview locally

**Why:** the same checks as Problems and Preview, from the editor and terminal.

```
sf agent validate authoring-bundle --api-name <BundleName> --target-org af-dev
sf agent preview --authoring-bundle <BundleName> --target-org af-dev
```

In VS Code the same actions are on the right-click menu of the `.agent` file: **AFDX: Validate this Agent**, **AFDX: Preview This Agent** (choose Simulation or Live Test), **AFDX: Publish This Agent**.

Preview modes match the browser: simulated mode mocks the actions; live mode uses the real Flows in the org and needs the agent's user to be set in the script.

### 3.4 Exercise: the second-order bug

**Why:** first change made locally, and it fixes the weakness parked in lesson 2.

The problem: `eligibility_checked` and `return_id` last for the session. If the customer names a second order, the first order's results are reused.

Let the learner design the fix before offering anything. Hints, in order, only as needed:

1. Baseline first: reproduce it in preview with two order numbers in one session.
2. What does the flag really need to remember: "a check ran" or "a check ran for this order number"?
3. What should happen to the return ID when the order number changes?

No reference solution is given here; none has been tested yet. When a learner's solution passes the two-order test and the lesson 2 tests, record it in "Course feedback" so it can become the reference.

### 3.5 Publish and verify

**Why:** closes the loop: the local change becomes a new version in the org.

```
sf agent validate authoring-bundle --api-name <BundleName> --target-org af-dev
sf agent publish authoring-bundle --api-name <BundleName> --target-org af-dev
git commit -am "Reset eligibility state when the order number changes"
```

Open the agent in the browser Builder and confirm the new version is there. Re-run the lesson 2 tests in Preview.

## To decide while shaping this lesson

- Whether to also create a remote repository and push, or keep git local.
- Whether to introduce CLI-driven tests (`sf agent generate test-spec`) here or in the testing lesson.
- Whether the browser-based Agentforce Vibes IDE deserves a mention as a no-install alternative.

## Quiz bank

To be written after the lesson has been run once.
