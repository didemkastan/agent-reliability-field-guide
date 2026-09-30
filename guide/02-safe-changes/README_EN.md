# Safe Changes and Agent Automation

[🇹🇷 Türkçe](README.md)

## Problem

When an agent is asked to make a small change, it may modify other areas that are not needed to produce the requested result.

For example, while fixing a single link, the agent may also rewrite text in the same file or change unrelated files. The requested link may be correct, but the change has gone beyond the assigned scope.

## Core rule

> **Before making a change, define what may change and what must be preserved. Make the smallest change needed to achieve the requested result.**

## Change boundary

For a simple task, three fields may be enough:

- **CHANGE:** What should be changed.
- **PRESERVE:** What must remain unchanged.
- **DO NOT:** Actions that are outside the task scope.

For example:

```text
CHANGE:
- Replace the old link in README.md with the new one.

PRESERVE:
- Other text
- Heading order
- File structure

DO NOT:
- Do not change other files.
- Do not rewrite unrelated text.
```

These boundaries do not need to become a long list for every task. For a small change, identifying only the critical constraints is usually enough.

## Make the smallest necessary change

An agent should not add unrelated improvements to the same change.

If one line is enough to correct a typo, rewriting the paragraph or reformatting the file is unnecessary. If the agent notices another issue, reporting it separately is safer than silently expanding the task.

## Example

An agent is asked to replace one link in a README file.

Expected change:

```text
README.md
- 1 link changed
```

At the end of the task, three files have changed:

```text
README.md
config.yaml
package.json
```

Even if the link is now correct, the changes to `config.yaml` and `package.json` were not part of the task. The unexpected changes should be reviewed and reverted if they are not required.

## How to check the scope

At the end of the task, compare the expected change with what actually changed:

```text
EXPECTED_FILES_CHANGED: 1
ACTUAL_FILES_CHANGED: 1

EXPECTED:
- README.md

ACTUAL:
- README.md

UNEXPECTED_CHANGES:
- none
```

File count alone is not enough. A protected section may have changed inside an expected file, so review the changed lines or diff when possible.

## Usable prompt

> **Before starting, define the scope of the requested change: identify which files or sections may change and what must be preserved. Make only the smallest change needed to complete the assigned task. Do not modify unrelated files, text, configuration, dependencies, or existing working behavior.**
>
> **If you notice another issue outside the scope while working, do not fix it automatically; report it separately. If completing the task requires going beyond the scope defined at the start, state this before continuing and explain why the additional change is necessary.**
>
> **When the task is complete, compare the actual changes with the original scope. Check which files and sections changed and, when possible, review the diff for unexpected modifications. Confirm that protected areas remained unchanged. If anything changed outside the scope, do not treat it as part of the task; report it clearly and revert it if it is not required.**
>
> **Finish with a short record:**
> - **CHANGED:** What was actually changed
> - **PRESERVED:** What was checked and confirmed unchanged
> - **UNEXPECTED_CHANGES:** Unexpected changes; use `none` if there are none
> - **SCOPE_STATUS:** Whether the change stayed within the assigned scope

## Where should the prompt go when using multiple agents?

When several agents work on the same project, the user should not have to say **“read the rules first”** at the start of every task. Shared rules should be defined once, and each agent should be connected to them through its project startup mechanism.

The shared rules can be kept in one **canonical project instruction**. For example:

```text
PROJECT/
├── AGENTS.md          ← shared rules for all agents
├── [Agent A entry]    ← connected to the AGENTS.md rules
├── [Agent B entry]    ← connected to the AGENTS.md rules
├── [Agent C entry]    ← connected to the AGENTS.md rules
└── ...
```

The important point is not the file name itself but having **one shared source of rules**. Different agent tools may load project instructions from different files or settings, so each agent's startup mechanism should be connected to the shared rules.

> **The presence of a file in the project does not mean every agent reads it automatically. During setup, confirm that each agent actually loads the shared rules.**

Once this connection is configured correctly, the shared rules do not need to be repeated in every task prompt. Only information specific to the current task is supplied:

```text
TASK:
- Fix the old link in README.md.

CHANGE:
- The relevant link

PRESERVE:
- Other text and headings

DO NOT:
- Do not change other files.
```

This creates a clear separation:

```text
SHARED / PERSISTENT
AGENTS.md
- safe-change rules
- verification rules
- general preservation constraints

TASK-SPECIFIC
TASK / CHANGE / PRESERVE / DO NOT
- boundaries of the current task only
```

A startup check can also be recorded in a multi-agent workflow:

```text
INSTRUCTION_SOURCE: AGENTS.md
INSTRUCTION_VERSION: v1.3
INSTRUCTIONS_LOADED: YES
```

This makes the instruction source visible instead of merely assuming that the agent had access to the rules. When the rules change, additional information such as a version or hash can also be used.

In an automated multi-agent system, the orchestrator can handle this step by passing the shared rules and task-specific boundaries to the agent performing the work. If the task is handed to another agent, the relevant scope and protected areas travel with the handoff record.

### Example: task flow with ChatGPT, Codex, Claude, Gemini, and GitHub

Several agents can work together on the same software project with different roles. For example:

- **ChatGPT — Orchestrator:** Analyzes the request, prepares the task, defines its boundaries, and routes the next step to the appropriate agent.
- **Codex — Developer:** Makes the required code change on the current project version and records what changed.
- **Claude — Review / correction:** Independently reviews the change and feeds errors, omissions, or out-of-scope changes back into the correction loop.
- **Gemini — Tester / QA:** Independently tests the implemented solution and looks for functional, regression, or interface issues.
- **GitHub — Shared workspace:** It is not an agent. It is the canonical workspace for the current code, change history, shared rules, and task records.

Example flow:

```text
                 ChatGPT
               ORCHESTRATOR
       TASK / CHANGE / PRESERVE / DO NOT
                     │
                     ▼
                   Codex
                 DEVELOPER
                 code change
                     │
                     ▼
                   GitHub
              SHARED WORKSPACE
        code + version + task records
                     │
                     ▼
                   Claude
             REVIEW / CORRECTION
                     │
          if needed, correction loop
                 back to Codex
                     │
                     ▼
                   GitHub
                     │
                     ▼
                   Gemini
                 TESTER / QA
```

These roles do not have to be fixed; they can change according to the project and task. What matters is that each agent's responsibility, working version, and destination for its result are explicit.

#### An agent without direct access to the shared workspace

Not every agent may be able to connect directly to GitHub. This does not mean that the agent cannot participate in the project.

> **An agent does not need to be connected to GitHub to participate in the project.**

For example, suppose the Gemini tester in the workflow above does not have direct GitHub access. The orchestrator can provide the current information required for the test as a task package:

```text
TASK_ID:
INPUT_VERSION:

TASK:
FILES / CONTEXT:

RULES:
PRESERVE:
DO NOT:

EXPECTED_OUTPUT:
LIMITATIONS:

RETURN_TO:
NEXT_AGENT:
NEXT_ACTION:
```

The package states which version the agent is working from, which files or information it can see, what it should test, what must be preserved, and what it cannot access. The external agent returns its result with the same `TASK_ID` and `INPUT_VERSION`.

### Filled example: tester without GitHub access

Suppose we want to check whether a change to a screen transition has broken existing behavior.

Only the **current-version** files required for the task are attached for Gemini:

```text
navigation.py
settings_screen.py
tests/test_navigation.py
test_output.txt
```

Selecting only the files needed for the task, instead of sending the entire repository, limits the scope and helps prevent the agent from making assumptions about areas it cannot see.

Task package for Gemini:

```text
TASK_ID: UI-024
INPUT_VERSION: commit abc123

TASK:
Review the behavior when returning from the Settings screen.
Check for regression risk in existing screen transitions.

FILES / CONTEXT:
- navigation.py
- settings_screen.py
- tests/test_navigation.py
- test_output.txt

RULES:
- Base the assessment only on the supplied files and test output.
- Keep observations separate from interpretations.
- State checks that could not be performed.

PRESERVE:
- Existing working screen transitions
- Navigation behavior outside Settings

DO NOT:
- Modify the files.
- Treat unseen repository content as reviewed.
- Assume access to the current GitHub state.

EXPECTED_OUTPUT:
- Findings
- File/line or test evidence supporting each issue
- Recommended next action
- Checks performed and skipped

LIMITATIONS:
- No direct GitHub access.
- Only the attached files and test output are visible.

RETURN_TO: ChatGPT / Orchestrator
NEXT_AGENT: Codex
NEXT_ACTION:
Validate the finding against the current GitHub version.
If it is still valid, make the smallest necessary fix.
```

#### Example prompt for Gemini

> **Review the attached task package and files. Preserve TASK_ID and INPUT_VERSION exactly in your response. Base your assessment only on the supplied files and test output. Write OBSERVED first and INTERPRETED separately. If you find an issue, identify the file, section, or test result that supports it. Put tests you could not run and areas you could not access under SKIPPED_CHECKS. Do not modify the files. Return the result using this structure:**
>
> ```text
> TASK_ID:
> INPUT_VERSION:
> 
> OBSERVED:
> INTERPRETED:
> 
> FINDINGS:
> EVIDENCE:
> 
> VERIFIED:
> SKIPPED_CHECKS:
> 
> RECOMMENDED_ACTION:
> RETURN_TO:
> NEXT_AGENT:
> ```

For example, Gemini might return:

```text
TASK_ID: UI-024
INPUT_VERSION: commit abc123

OBSERVED:
The return action in settings_screen.py recreates selected_theme.
test_navigation.py does not check this state.

INTERPRETED:
There is a risk that the selected theme is lost when returning
from the Settings screen.

FINDINGS:
- State preservation in the return flow should be checked.
- The current test does not cover this scenario.

EVIDENCE:
- settings_screen.py: return action
- tests/test_navigation.py: no corresponding state assertion

VERIFIED:
- The three supplied source files and test output were reviewed.

SKIPPED_CHECKS:
- The application was not executed.
- Other repository files were not reviewed.

RECOMMENDED_ACTION:
Reproduce the behavior on the current GitHub version.
If confirmed, make the smallest fix and add a regression test.

RETURN_TO: ChatGPT / Orchestrator
NEXT_AGENT: Codex
```

There are two different routing fields here: **`RETURN_TO`** identifies who receives Gemini's result first, while **`NEXT_AGENT`** identifies which agent should continue the task after the routing checks are complete.

Two operating modes should be kept separate in this example.

#### Returning the result from an agent without GitHub access

In this example, Gemini is not directly connected to the shared workspace in GitHub. The user therefore provides Gemini with the task package and the files required for the task. When Gemini completes the test, the user passes its result back to ChatGPT / the orchestrator:

```text
Gemini completes the test
        ↓
User gives the Gemini result to ChatGPT
        ↓
ChatGPT / Orchestrator
reads TASK_ID and INPUT_VERSION
        ↓
checks CURRENT_VERSION in GitHub
        ↓
compares INPUT_VERSION ↔ CURRENT_VERSION
        ↓
prepares a current task package for Codex
        ↓
User gives the task package to Codex
        ↓
Codex validates the finding
        ↓
if needed, minimum fix + test
        ↓
GitHub
```

In this mode, ChatGPT is not expected to reproduce Gemini's technical finding itself. ChatGPT performs the **handoff checks**: it identifies the task and version associated with the finding, compares them with the current GitHub state, preserves the required context, and prepares a safe task package for Codex.

For example, the Gemini result can be given to ChatGPT with this prompt:

> **The result below came from a tester agent without direct GitHub access. First check TASK_ID and INPUT_VERSION. Retrieve the current GitHub branch/commit and record it as CURRENT_VERSION. If INPUT_VERSION and CURRENT_VERSION do not match, do not route the result to Codex as a direct implementation task; check which relevant files changed and mark the finding for revalidation. If they match, prepare the next task package for Codex without losing the finding, evidence, SKIPPED_CHECKS, PRESERVE, or LIMITATIONS. Do not change code.**
>
> **Tester result:**
>
> ```text
> TASK_ID: UI-024
> INPUT_VERSION: commit abc123
> FINDING: The selected theme may be lost in the return flow.
> EVIDENCE: settings_screen.py return action; no state assertion in the current test.
> SKIPPED_CHECKS: Application not executed; other repository files not reviewed.
> RETURN_TO: ChatGPT / Orchestrator
> NEXT_AGENT: Codex
> ```

ChatGPT might prepare an output like this:

```text
TASK_ID: UI-024
TESTER_INPUT_VERSION: commit abc123
CURRENT_VERSION: commit abc123
VERSION_STATUS: MATCH

SOURCE_FINDING:
The selected theme may be lost in the return flow.

EVIDENCE:
settings_screen.py return action;
no state assertion in the current test.

SKIPPED_CHECKS:
- Application was not executed by the tester.
- Other repository files were not reviewed by the tester.

NEXT_AGENT: Codex
NEXT_ACTION:
Validate the finding against the current code.
If confirmed, make the smallest necessary fix and complete the test.
```

This output is the **task package to give to Codex**. The external finding therefore does not turn directly into a “fix this” command; it first passes through version and context checks.

#### Giving ChatGPT's prepared task to Codex

In a manual or semi-automated workflow, the user gives the task package above to Codex. The instruction can be:

> **The task package below was prepared by the orchestrator. First check the GitHub branch/commit you can access and compare it with CURRENT_VERSION in the package. If they do not match, reassess the task against the current version before making any change. If they match, independently validate the external tester's finding against the current code and tests. If the issue is confirmed, follow AGENTS.md and the PRESERVE boundaries and make the smallest necessary fix. If it is not confirmed, do not change the code; report the evidence instead.**
>
> **At the end, return CURRENT_VERSION, CHANGED, PRESERVED, VERIFIED, SKIPPED_CHECKS, and SCOPE_STATUS.**

#### What does the trigger do in a fully automated handoff?

In a fully automated system, the user does not need to perform the two transfers above. A **triggering mechanism** works alongside the orchestrator.

```text
Gemini produces TASK_COMPLETE
        ↓
Trigger delivers the result to the orchestrator
        ↓
Orchestrator checks GitHub CURRENT_VERSION
        ↓
if version and scope are valid
        ↓
Codex task is created / started
        ↓
Codex result returns to the orchestrator
```

The trigger can be an API event, webhook, queue, CI/CD step, or a task-completion event provided by the agent platform. Regardless of technology, its job is to capture **“the previous agent finished”** and deliver the result to the orchestrator. The orchestrator then checks version, scope, and authority before creating the next task.

The **orchestrator** and the **trigger** are therefore not the same thing: the trigger starts the transition; the orchestrator controls what is passed, with which context, and to which agent.

The receiving Codex agent does not treat the external agent's result as established fact. Its job is to **revalidate it against the current project state**:

```text
Gemini finding
TASK_ID + INPUT_VERSION
        ↓
ChatGPT / Orchestrator
preserves context and boundaries
        ↓
Codex
checks CURRENT_VERSION
        ↓
revalidates the finding
        ↓
not confirmed ──→ no change; report result
        │
      confirmed
        ↓
minimum fix
        ↓
test / diff / scope check
        ↓
GitHub
```

This lets an agent without GitHub access contribute to the project without turning its finding directly into project truth or automatic authority to change the code.

```text
GitHub
shared workspace
     │
     ▼
ChatGPT / Orchestrator
prepares the current task package
     │
     ▼
Gemini
tests without direct GitHub access
     │
     ▼
TEST RESULT
TASK_ID + INPUT_VERSION
     │
     ▼
Orchestrator / connected agent
compares the result with the current version
     │
     ▼
GitHub
```

The result should not be applied to the project immediately. First compare the `INPUT_VERSION` given to the external agent with the current version in GitHub. If the project changed in the meantime, reassess the result against the current version.

This allows an agent without direct access to the shared workspace to participate in the project in a controlled way. What matters more than direct GitHub access is **current task context, version information, explicit authority boundaries, and structured handoff.**

For the workflow to advance automatically between agents, an **orchestrator or triggering mechanism** is still required. Giving multiple agents access to the same GitHub repository does not, by itself, cause one agent to start automatically when another finishes.

## Safe Agent Handoffs and Automation Setup

So far, we have seen what information should travel with a task when it moves from one agent to another. Now we can look at how the same handoff can happen without requiring the user to carry every message manually.

The goal of this section is not simply to create a few workflow files. First, it is important to understand **how agents see each other's work** and then **how GitHub starts the next agent**.

### How do agents see each other's work?

The most important point is:

> **A change made by one agent does not automatically appear in another agent's session. GitHub is the shared point between them.**

For example, if Codex changes a file only inside its own working environment, Claude or ChatGPT cannot see that change yet.

The change first needs to reach GitHub:

```text
Codex changes a file
        ↓
the change is committed
        ↓
it is sent to GitHub
        ↓
a branch or Pull Request is updated
        ↓
GitHub now knows about the change
```

A **local change and a GitHub change are not the same thing.**

If Codex changed a file but has not sent the change to GitHub, there is no GitHub event to trigger.

Once the change reaches GitHub, another agent such as Claude can be started. The new agent works from the current repository, branch, commit, or Pull Request stored in GitHub.

```text
Codex
  ↓
sends the change to GitHub
  ↓
GitHub stores the current state
  ↓
Claude is started
  ↓
Claude reads the current change from GitHub
```

Codex does not need to send a direct message to Claude. **GitHub becomes the shared workspace and Source of Truth.**

### GitHub access and automatic execution are not the same thing

Giving an agent access to GitHub does not mean that it is continuously watching GitHub and immediately sees every change.

There are two separate capabilities:

```text
GITHUB ACCESS
= the agent can read the repository when needed
  and perform actions allowed by its permissions

AUTOMATIC EXECUTION
= the system can start the agent automatically
  when a defined event occurs
```

For example, giving ChatGPT access to GitHub in a chat does not mean ChatGPT automatically runs after every commit.

Likewise, simply having a Codex or Claude connection does not create an automated agent handoff.

GitHub Agentic Workflows can directly select GitHub Copilot, Claude Code, OpenAI Codex, and Google Gemini as the **engine** that runs a workflow. If the ChatGPT product is used as a separate orchestrator, it needs its own programmatic integration so the automation can invoke it.

### What does a GitHub “event” mean?

An event is a specific GitHub change that automation can react to.

For example:

```text
Pull Request opened
or
new commit added to a PR
or
label applied
or
workflow started manually/automatically
        ↓
GitHub Actions checks the matching rule
        ↓
the matching workflow starts
```

If Codex sends a change to GitHub and opens a Pull Request, GitHub records a **new PR opened** event. If a rule is listening for that event, Claude's review task can start automatically.

### The basic parts of the automation

Think of the system like this:

```text
1. SHARED STATE
current code / PR / commit in GitHub

        ↓

2. TRIGGER
“Something changed; start the workflow.”

        ↓

3. ROUTING
“Which task should run next?”

        ↓

4. AGENT
Codex / Claude / Gemini performs the task

        ↓

5. RESULT
finding, commit, PR, test result, or task record

        ↓

6. VERIFICATION
Is the result actually correct?

        ↓

7. NEXT HANDOFF
Start the next workflow if needed
```

There is no assumption of an invisible conversation between agents. Each new run uses its assigned task and the current state in GitHub.

### Example flow: Codex → Claude → Codex

In this example, Codex changes the code and Claude reviews the change. If Claude finds a problem, the task returns to Codex.

```text
Task
  ↓
Codex makes the change
  ↓
change is sent to GitHub
  ↓
Pull Request opens
  ↓
Claude review starts
  ↓
 ┌─────────────────────┐
 │                     │
problem found       no problem
 │                     │
 ▼                     ▼
back to Codex       run tests
 │
 ▼
Codex validates the finding
and fixes it if needed
 │
 ▼
PR is updated
 │
 ▼
Claude reviews again
```

The user no longer needs to carry messages such as **“Codex is done; now start Claude”** or **“Claude found a problem; send it back to Codex.”**

For that flow to be truly automatic, however, the setup below must exist.

### 1. What you need before setup

Before using GitHub Agentic Workflows, make sure the basic requirements are available:

```text
Repository
→ you have write access

GitHub Actions
→ enabled for the repository

GitHub CLI
→ installed and authenticated

Agent account
→ access to the Codex / Claude / Gemini / Copilot engine you will use

Authentication
→ required API key or supported authentication method is ready
```

Commands beginning with `gh` are **not entered on the GitHub website or inside a project file. They run in a command-line window on your computer.** On Windows, open **PowerShell** or **Windows Terminal**. On macOS/Linux, open **Terminal**.

GitHub CLI is the tool that lets us perform GitHub operations from this command-line window. First, open Terminal/PowerShell and check whether GitHub CLI is installed and connected to your account:

```bash
gh --version
gh auth status
```

The first command checks whether GitHub CLI is installed. The second shows its GitHub authentication status.

If needed, authenticate with repository and workflow scopes in the same Terminal/PowerShell window:

```bash
gh auth login --scopes repo,workflow
```

### 2. Prepare the repository for Agentic Workflows

Install the GitHub Agentic Workflows extension in the **same Terminal/PowerShell window**:

```bash
gh extension install github/gh-aw
gh aw init
```

The first command installs the Agentic Workflows extension. The second prepares the current repository for Agentic Workflows. Before running them, make sure Terminal/PowerShell is currently inside the repository folder you want to work with.

Agent workflow sources are stored under `.github/workflows/`.

For example:

```text
.github/workflows/
├── review.md
├── review.lock.yml
├── fix.md
└── fix.lock.yml
```

The `.md` file is the readable workflow definition.

Its top section defines **when the task starts, which agent is used, and which permissions are available**. The body contains the natural-language task for the agent.

The `.lock.yml` file is the compiled workflow that GitHub Actions runs.

When workflow settings change, run the following from **Terminal/PowerShell opened in the repository folder**:

```bash
gh aw compile
```

This generates the `.lock.yml` version that GitHub Actions runs. Then send both the source `.md` and generated `.lock.yml` to GitHub.

If the workflow exists only on your computer and has not been sent to GitHub, GitHub cannot run it.

### 3. Do not put agent keys in the code

If an AI engine such as Codex, Claude, or Gemini uses an API key, the real key should **not be written into repository files, source code, or normal workflow text.** Store it in GitHub's **Actions secrets** area.

#### Where does the key come from?

Create an API key in the account for the AI service you use:

```text
Codex   → API key from your OpenAI account
Claude  → API key from your Anthropic account
Gemini  → API key from Google AI Studio
```

Then store that value in GitHub as a secret.

#### Where do you enter it in GitHub?

Open the repository where the automation will run on the GitHub website, then follow:

```text
Repository
   ↓
Settings
   ↓
Secrets and variables
   ↓
Actions
   ↓
New repository secret
```

The page has two important fields:

```text
Name
→ the name the workflow uses to find the key

Secret
→ the real API key obtained from the AI provider
```

For Claude:

```text
Name:
ANTHROPIC_API_KEY

Secret:
[your real key from the Anthropic account]
```

For Codex:

```text
Name:
OPENAI_API_KEY

Secret:
[your real key from the OpenAI account]
```

For Gemini:

```text
Name:
GEMINI_API_KEY

Secret:
[your real key from Google AI Studio]
```

Finally, select **Add secret**.

The current basic secret names for GitHub Agentic Workflows are:

```text
Codex   → OPENAI_API_KEY or CODEX_API_KEY
Claude  → ANTHROPIC_API_KEY
Gemini  → GEMINI_API_KEY
```

Codex accepts both `CODEX_API_KEY` and `OPENAI_API_KEY`; if both are present, `CODEX_API_KEY` takes precedence.

#### How does the workflow use the key?

The basic flow is:

```text
Create a key with the AI provider
        ↓
store it in GitHub Actions Secrets
        ↓
workflow runs
        ↓
GitHub securely supplies the secret during the run
        ↓
the AI engine authenticates
```

The real key therefore does not appear in normal repository files.

Do **not** put the real key in:

```text
AGENTS.md
normal workflow text
task packages
README files
source code
commit messages
Issue / Pull Request comments
```

Keys can also be stored from Terminal/PowerShell, but this guide uses the GitHub web interface as the primary path for beginners.

> **GitHub Copilot can work differently:** With organization-backed GitHub Copilot usage, the `copilot-requests: write` permission can remove the need for a separate AI-provider API key. Recheck the current authentication method for the selected engine and account setup.

The API key here is used to **authenticate to the AI service**. Permissions to read repository files, update a PR, add comments, or start another workflow are controlled separately through GitHub permissions and workflow configuration.

### 4. How does Claude see the change after Codex finishes?

Codex saying **“done”** is not enough.

The change must reach GitHub:

```text
Codex made the change
        ↓
commit / branch / PR sent to GitHub
        ↓
GitHub stored the current version
        ↓
Claude workflow started
        ↓
Claude read the current PR code and changes
```

For a Pull Request workflow, GitHub Agentic Workflows checks out the relevant repository for the run and, for a PR event, can work from the PR head context.

Claude is not reading Codex's memory. **It is reading the recorded change in GitHub.**

Example trigger:

```markdown
---
on:
  pull_request:
    types: [opened, synchronize]

engine: claude

permissions:
  contents: read
  pull-requests: read

safe-outputs:
  add-comment:
---

# Review

Review the PR against the task boundaries.

First record the current PR HEAD commit as CURRENT_VERSION.

Check:
- Was the requested change implemented?
- Were PRESERVE areas kept unchanged?
- Were unrelated files modified?
- Are tests or verification steps missing?

Report findings with evidence.
```

Here:

```text
opened
= run when the PR is first opened

synchronize
= run again when a new commit is added to the PR
```

### 5. Recheck the version at every handoff

Do not assume that the version Claude reviewed is still the version Codex will later modify.

Another commit may have arrived in between.

At minimum, compare:

```text
Claude reviewed
INPUT_VERSION: abc123

        ↓

Codex received the task
CURRENT_VERSION: abc123

        ↓

MATCH
→ continue with the finding on the current version
```

If:

```text
INPUT_VERSION: abc123
CURRENT_VERSION: def456
```

Codex should not apply the old finding directly. It should first confirm that the finding is still valid on the new version.

This check should remain part of the automation.

### 6. How does Claude's result start Codex?

Two different methods should not be confused.

#### Method A — Start the next workflow directly

For an explicit agent-to-agent automation chain, one workflow can use **dispatch-workflow** to start an allowed worker workflow.

The idea is:

```text
Claude finishes review
        ↓
problem found
        ↓
start fix workflow
        ↓
Codex runs
```

For example, the review workflow can be allowed to dispatch only the `fix` workflow:

```yaml
safe-outputs:
  dispatch-workflow:
    workflows: [fix]
    max: 1
```

The `fix` workflow accepts `workflow_dispatch`.

Pass task information such as `TASK_ID`, PR number, reviewed commit, and finding into the next workflow.

#### Method B — Use a label as state or a command

A GitHub label can also represent a state:

```text
agent:fix-required
= correction required
```

With `label_command`, a specific label can act like a one-shot command that starts a workflow.

There is an important detail, however: some writes performed with GitHub's default `GITHUB_TOKEN` do not start new workflow or CI runs. GitHub uses this behavior to prevent accidental automation loops.

So do not assume:

> **“Claude added a label, therefore Codex will definitely start.”**

If labels, agent-created PRs, or agent-generated commits are expected to trigger another workflow, the token and trigger path must be configured accordingly.

Agentic Workflows can also use a suitable CI-trigger credential when safe-output PR creation or PR-branch pushes need to trigger CI. Another option is to use `dispatch-workflow` explicitly for agent-to-agent routing.

A clear beginner model is:

```text
SHOW THE STATE
→ comment / label

START THE NEXT AGENT
→ dispatch-workflow
```

This keeps **showing state** separate from **actually starting another agent**.

### 7. Codex should not blindly apply Claude's finding

When Claude reports a problem, do not simply tell Codex:

> “Claude said so; fix it.”

Codex should first check the current GitHub state:

```text
1. Which PR am I working on?
2. Which commit did Claude review?
3. What is the current PR HEAD commit?
4. Is the finding still valid?
5. Which files am I allowed to change?
```

Only then should it make the smallest necessary correction.

Example task:

```text
TASK_ID: UI-024
INPUT_VERSION: abc123
CURRENT_VERSION: abc123

FINDING:
- State is not preserved in the return flow.

PRESERVE:
- Other screen transitions

ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

NEXT_ACTION:
- Validate the finding.
- If valid, make the smallest correction.
- Run the relevant test.
```

### 8. How does Claude run again after the fix?

When Codex sends the fix to the PR branch, the code in GitHub changes.

But **updating the code and definitely starting the next workflow are not the same thing.**

GitHub Actions limits new workflow chains caused by some events created with the default automation token.

So a fully automated loop should define the next transition explicitly:

```text
Codex made the correction
        ↓
PR updated
        ↓
review workflow explicitly started again
        ↓
Claude read the current commit again
```

Two approaches are possible:

```text
A) Dispatch the review workflow from the Codex result

or

B) Configure a suitable GitHub identity that allows
   the PR update to trigger the intended CI/workflow
```

For a first setup, **A**, explicit workflow-to-workflow routing, is easier to understand and debug.

### 9. Should the test stage use an AI agent or a normal automated test?

You do not need to start another AI agent for every test. First ask:

> **Can a program determine the result of this check clearly, or does someone need to interpret the result in context?**

Separate the checks into two types.

#### 1. Checks with a clear result

Some checks have an objective result. For example:

```text
Did the unit test pass?
→ YES / NO

Did the application build successfully?
→ YES / NO

Are there lint errors?
→ YES / NO

Did the type check pass?
→ YES / NO
```

An AI agent does not need to read the code and decide these results. Existing test commands, scripts, or **GitHub Actions** can run these checks automatically.

For example, if the project already uses:

```bash
pytest
```

GitHub Actions can run the same command automatically and route the workflow according to whether it succeeds.

The flow becomes:

```text
Codex made the change
        ↓
sent to GitHub
        ↓
normal automated test runs
        ↓
 ┌──────────────────────┐
 │                      │
TEST PASSED          TEST FAILED
 │                      │
 ▼                      ▼
next check           correction task for Codex
```

The **normal automated test** here is not AI. It is the project's existing test tool being run automatically by GitHub Actions.

#### 2. Checks that require interpretation

Some questions cannot be answered by a test command returning only `PASS` or `FAIL`.

For example:

```text
Was the behavior requested by the user actually implemented?

Did the change leave the assigned scope?

Was existing working behavior changed unnecessarily?

Could the code work technically while misunderstanding the request?

Does the screen or user flow match the expected behavior?
```

For these checks, an **AI agent such as Claude, Gemini, or another suitable agent** can read the context and evaluate the result.

Normal tests and AI agents are therefore not substitutes for each other. A reliable workflow will often use both:

```text
Code change
      ↓
NORMAL AUTOMATED CHECKS
unit test / build / lint / type check
      ↓
pass
      ↓
AI REVIEW
was the request implemented correctly?
was scope preserved?
is there a contextual problem?
      ↓
verification
```

The order can vary by project. For example, running cheap and fast automated tests before an AI review can avoid spending AI usage on a change that does not even build.

In short:

```text
Can a command determine the answer clearly?
        ↓
YES → normal test / script / GitHub Actions

Does the check require interpretation, context, or judgment?
        ↓
YES → AI agent
```

The goal is not to remove AI agents from testing. It is to **avoid using AI for work that existing tools can determine exactly, and use AI where interpretation or contextual evaluation is valuable.**

### 10. Trigger, orchestration, and agent are different roles

In simple terms:

```text
TRIGGER
= “start now”

ORCHESTRATION RULE
= “who should start now, and with which task?”

AGENT
= “do the assigned task”

GITHUB
= “store the shared current state and run records”
```

For example:

```text
PR opened
        ↓
trigger started review workflow
        ↓
Claude reviewed
        ↓
routing rule selected fix workflow
        ↓
Codex received the correction task
```

If ChatGPT is also used as an orchestrator, there must be a real integration that allows the automation to invoke ChatGPT. GitHub access alone does not provide that behavior.

### 11. Give agents only the permissions they need

Once automation is enabled, an agent may act without waiting for the user. Apply **least privilege**.

GitHub Agentic Workflows keeps agent execution read-oriented by default. Write operations can be handled through **safe outputs** in a separate controlled step.

For example:

```text
Claude
→ read code
→ review PR
→ request comment

Codex
→ prepare correction only in allowed files
→ send it to the PR branch through a controlled output
```

File boundaries can also be defined:

```text
ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

PROTECTED:
- AGENTS.md
- .github/**
- dependency / package files
```

Do not automatically allow an agent to modify workflow, security, instruction, or dependency files that are not needed for its task.

### 12. Prevent the same task from running twice

If two runs start for the same PR or task at the same time, they may invalidate each other's results.

GitHub Actions and Agentic Workflows can use concurrency controls to limit overlapping runs.

The idea is:

```text
UI-024 is running
        ↓
same task arrives again
        ↓
old/new run policy is checked
        ↓
conflicting changes are not applied at the same time
```

For PR-based agentic workflows, controls can also cancel an older run after a newer commit makes it stale.

### 13. Do not continue the chain after a failed step

**“The workflow started”** and **“the workflow succeeded”** are not the same thing.

If Claude could not run, an API key is invalid, required data could not be read, or Codex failed to send the correction to GitHub, the next step should not treat the previous result as successful.

```text
WORKFLOW STARTED
        ↓
did the task finish?
        ↓
was the required output produced?
        ↓
did verification pass?
        ↓
YES → next stage
NO  → stop / record failure
```

A failed run can first be inspected from the **Actions** tab on the GitHub website.

For a more detailed check, you can also use **Terminal/PowerShell** on your computer. Open it in the repository folder and run:

```bash
gh aw logs
```

to view Agentic Workflows runs. Find the **RUN_ID (run number)** for the run you want to inspect, then use:

```bash
gh aw audit <RUN_ID>
```

and replace `<RUN_ID>` with the actual run number. For example, if the run number is `123456`:

```bash
gh aw audit 123456
```

These commands are not used to create the automation. They are used to **check how an existing automation ran and investigate problems when something goes wrong**.

### 14. Put a limit on automation cost

An agent run should not be treated as a free and unlimited operation.

Agentic workflows may consume both GitHub Actions runtime and AI-provider inference.

Avoid starting an AI agent when a normal deterministic check can do the job:

```text
Can a deterministic check answer this?
        ↓
YES → normal test / script / GitHub Actions
NO  → use an AI agent if interpretation is needed
```

Agentic Workflows also supports per-run AI usage limits and usage inspection.

For example:

```yaml
max-ai-credits: 500
```

Choose a real limit based on the model and expected workload rather than copying the example value blindly.

### 15. Automate when the human should be notified

The goal of full automation is not to remove the human from the system completely. The goal is to let agents handle routine transitions while **making situations that require a decision or intervention visible to the right person.**

The orchestration flow should therefore include a **human notification / intervention point**.

For example:

```text
Agent is working
        ↓
normal, verified result
        ↓
automation continues

BUT

version mismatch
or
verification failed
or
the task needs to leave its allowed scope
or
more authority is required
or
the retry limit was reached
        ↓
STOP AUTOMATION
        ↓
notify the human
        ↓
wait for decision / approval
```

The exact notification conditions depend on project risk. Good candidates include:

- the task completed successfully and the final result is ready;
- a workflow or agent failed repeatedly;
- `INPUT_VERSION` does not match `CURRENT_VERSION`;
- an agent needs to leave the assigned scope;
- a new file, service, or higher permission is required;
- verification is uncertain or failed;
- the automatic retry limit is exhausted;
- a security-sensitive situation requires a human decision.

A notification should not merely say **“something failed.”** It should contain enough information for the person to make a decision:

```text
TASK_ID
STATUS
CURRENT_VERSION
WHAT_HAPPENED
EVIDENCE
WHAT_WAS_TRIED
WHAT_NEEDS_HUMAN_DECISION
SAFE_NEXT_OPTIONS
```

For example:

```text
TASK_ID: UI-024
STATUS: HUMAN_REVIEW_REQUIRED
CURRENT_VERSION: def456

WHAT_HAPPENED:
The repository version changed after Claude's finding.

EVIDENCE:
INPUT_VERSION: abc123
CURRENT_VERSION: def456

WHAT_NEEDS_HUMAN_DECISION:
Should the task be restarted against the new version?
```

Where the notification is sent depends on the system. A GitHub Issue, Pull Request comment, or another configured notification channel can be used. Email, Slack, or another external channel requires a separate integration that can access that channel.

Rather than notifying a human about every small agent action, notifications are more useful at **completion, stop, failure, and decision-required thresholds**. Too many notifications can hide the important ones.

Orchestration should therefore answer not only:

```text
"Which agent is next?"
```

but, when needed:

```text
"Should automation stop here?"
"Should the human be notified?"
"Is human approval required before continuing?"
```

### 16. Start with one small automated handoff

Do not connect every agent at once.

A good first target is:

```text
Codex sent the change to GitHub
        ↓
PR opened
        ↓
Claude started reviewing automatically
```

Once that transition is reliable, add the next steps one at a time:

```text
Claude → Codex correction
Codex → Claude re-review
Review → tests
Tests → verification
```

At every handoff, preserve at least:

```text
TASK_ID
INPUT_VERSION
CURRENT_VERSION
SOURCE_AGENT
NEXT_AGENT
FINDINGS
EVIDENCE
PRESERVE
SKIPPED_CHECKS
NEXT_ACTION
```

The goal is not only to run agents in sequence.

> **Safe automation should verify that the right task moves to the right agent with the right version and the right boundaries.**

> **Note:** GitHub Agentic Workflows is in public preview at the time of writing. Recheck the current GitHub documentation for engine, trigger, permission, safe-output, and authentication options during setup.

## When to use it

This method is especially useful for limited fixes to existing files or code, when working parts must be preserved, or when an agent should touch only specific files.

A detailed change-boundary record may not be necessary for small experiments that are easy to discard. For example, when comparing a few text formats in an empty test file, defining protected lines is usually unnecessary.
