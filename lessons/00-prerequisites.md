# Lesson 0: Prerequisites

**Goal:** the learner has their own Salesforce org with Agentforce switched on, and a local toolchain that can talk to it.

**Last verified:** steps 0.1 to 0.3 in a Developer Edition org on 2026-10-09. Steps 0.4 to 0.6 are taken from Salesforce documentation and have not been run end to end during course development. Verify commands against `reference/sources.md` before giving them, and record what the learner actually saw.

**Time:** 45 to 60 minutes, mostly waiting for sign-up email and installs.

Nothing is shared between learners. Each learner signs up for their own org and installs their own tools. Don't offer an org, a login or a project from anyone else.

## 0.1 Get your own org

**Why:** every later step needs an org where the learner is admin and can break things safely.

1. Go to `developer.salesforce.com/signup` and sign up for a free Developer Edition.
2. Username must look like an email address and be unique across all of Salesforce. It does not have to be a real mailbox. The email field does: the activation link goes there.
3. Open the activation email, verify the account, set a password.
4. Save the login URL and username in a password manager.

Expect: the learner lands on the org's home page. The URL contains `-dev-ed`.

Teach here, briefly, because learners mix these up:

| Org type | What it is |
|---|---|
| Production | A company's live org. Real data, paid licenses |
| Sandbox | A copy of one production org, for development and testing |
| Developer Edition | A free standalone org with no parent production. This is what the course uses |
| Scratch org | A disposable org created from a config file, for automation |

Learners often call the Developer Edition "my sandbox". It isn't one, and the difference matters at login: tools ask "Production or Sandbox?" and a Developer Edition logs in as **Production**.

Things to tell the learner:

- The sign-up terms say Salesforce may limit the AI features and may terminate an org that has been inactive for 45 days. Log in now and then.
- An older Developer Edition from before Agentforce was included may not have the features. If Agentforce Studio is missing after step 0.2, sign up for a new org.
- Never use an employer's or client's org for this course.

Record the org domain and sign-up date in `PROGRESS.md`.

## 0.2 Switch on Einstein and Agentforce

**Why:** agents can't be created until the generative AI layer and Agentforce are enabled.

1. Gear icon (top right) → **Setup**.
2. In Quick Find type `Einstein Setup`. Set **Turn on Einstein** to On. Refresh the browser tab.
3. In Quick Find type `Salesforce Go`. Search its features for `Agentforce`, select **Agentforce (Default)** → **Get Started** → **Turn On** → **Confirm**.

Expect: both toggles on. In a new Developer Edition they may already be on; then there is nothing to do.

## 0.3 Open Agentforce Studio

**Why:** confirms the org can build agents, and shows the learner where the course happens.

1. App Launcher (9-dot grid, top left) → search `Agentforce Studio` → open it → **Agents**.

Expect: an Agents list with a **New Agent** button. The list may already contain sample agents that came with the org (seen on 2026-10-09: three, from a sample resort app). Leave them alone; the course doesn't use them.

Ask for a screenshot. This is the first checkpoint.

## 0.4 Install the local tools

**Why:** from lesson 3 the agent is edited, validated and versioned on the learner's machine. Installing now means lesson 3 starts with learning, not with downloads.

Ask the learner to run these first and paste the output, so only missing tools get installed:

```
node -v
npm -v
git --version
sf --version
code -v
```

| Tool | Install if missing |
|---|---|
| Node.js and npm | LTS installer from `nodejs.org`. npm comes with it |
| git | `git-scm.com`, or the OS package manager |
| Salesforce CLI | `npm install -g @salesforce/cli` |
| VS Code | `code.visualstudio.com` |
| `code` command | In VS Code: Command Palette → "Shell Command: Install 'code' command in PATH" (macOS; on Windows the installer adds it) |
| Salesforce Extension Pack | VS Code → Extensions → search "Salesforce Extension Pack". It bundles the Agentforce DX extension |

Expect: all five commands print a version. Record the versions.

## 0.5 Connect the CLI to the org

**Why:** proves the toolchain end to end before any lesson depends on it.

1. Find the org's My Domain URL: it is the browser URL with `lightning.force.com` replaced by `my.salesforce.com`, without the path.
2. Run, with the learner's own domain:

   ```
   sf org login web --alias af-dev --set-default --instance-url https://YOUR-DOMAIN.my.salesforce.com
   ```

3. A browser window opens. Log in and click **Allow**.
4. Check:

   ```
   sf org list
   sf org display --target-org af-dev
   ```

Expect: the org is listed with alias `af-dev` and marked as default. Tell the learner not to paste the access token that `sf org display` prints; ask only whether the org shows as connected.

If the learner uses the VS Code command "SFDX: Authorize an Org" instead, the org type to choose is **Production**.

## 0.6 Decide where your own work will live

**Why:** this course repository is the lesson plan. The learner's agent project is theirs and belongs in a separate folder and repository, created in lesson 3.

1. Pick a parent folder for the future project, outside this repository.
2. Check git has an identity: `git config --global user.name` and `git config --global user.email`.

## Done when

- [ ] Own Developer Edition org, login saved
- [ ] Einstein and Agentforce on
- [ ] Agentforce Studio shows the Agents list with New Agent
- [ ] `node`, `npm`, `git`, `sf`, `code` all print a version
- [ ] Salesforce Extension Pack installed in VS Code
- [ ] `sf org list` shows the org as `af-dev`
- [ ] `PROGRESS.md` environment section filled in

No quiz for this lesson. Check the learner can answer one question: why does a Developer Edition log in as "Production"?
