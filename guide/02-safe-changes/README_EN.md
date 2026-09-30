# Safe Changes

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

The goal is simple:

```text
One agent finishes
        ↓
GitHub detects the event
        ↓
the next task starts
        ↓
the next agent works
```

The user no longer needs to keep saying **“Codex is done; now start Claude”** or **“Claude found a problem; send it back to Codex.”**

### First, understand the handoff

Connecting an agent to a GitHub repository does not automatically give it the ability to start another agent.

Something still needs to start the next step. In this example, **GitHub Actions** does that job.

When something happens in GitHub — for example, a Pull Request (PR) is opened, a new commit is added to a PR, or a specific label is applied — GitHub Actions can start the matching workflow.

```text
Something happens in GitHub
        ↓
GitHub Actions detects it
        ↓
the next task is selected
        ↓
the relevant agent runs
        ↓
the result returns to GitHub
        ↓
the next handoff can begin
```

Keep two ideas separate:

- **GitHub access:** lets an agent read project files and, if permitted, change them.
- **Automatic triggering:** starts the next task when a defined event occurs.

The agent service must also support being started from an external mechanism. That may be provided through an API, command-line tool, or ready-made GitHub integration.

GitHub **Agentic Workflows** is one option for building these flows on GitHub Actions. At the time of writing, it is in **public preview**, so the current GitHub documentation should be checked before using the setup examples below.

### Example flow: Codex → Claude → Codex

In this example, Codex changes the code and Claude reviews the change. If Claude finds a problem, the task returns to Codex.

```text
Task
  ↓
Codex makes the change
  ↓
Pull Request opens
  ↓
GitHub Actions starts
  ↓
Claude reviews the change
  ↓
 ┌─────────────────────┐
 │                     │
problem found       no problem
 │                     │
 ▼                     ▼
back to Codex       run tests
 │
 ▼
Codex fixes it
 │
 ▼
PR is updated
 │
 ▼
Claude reviews again
```

The important point is that **the agents do not need to send messages directly to each other.** The task state in GitHub and the workflow rules can decide which step comes next.

Now we can build the flow step by step.

### 1. Prepare the automation area in GitHub

First, the **GitHub CLI**, GitHub's command-line tool, should be installed and authorized for the repository.

If GitHub Agentic Workflows is being used, its extension can be prepared with:

```bash
gh extension install github/gh-aw
gh aw init
```

Automated agent tasks are stored under `.github/workflows/`.

For example:

```text
.github/workflows/
├── implement.md
├── implement.lock.yml
├── review.md
├── review.lock.yml
├── fix.md
└── fix.lock.yml
```

The Markdown files describe **which agent should do what and when**.

The settings at the top of a file define when the task starts, which agent is used, and what permissions it has. The normal text below those settings describes the task for the agent.

The `.lock.yml` files are compiled versions used by the system. When the source workflow changes, they can be regenerated with:

```bash
gh aw compile
```

### 2. Do not put agent keys in the code

If a service such as Codex, Claude, or Gemini requires an API key, do not write that value into normal repository files.

Store it in GitHub under:

**Settings → Secrets and variables → Actions**

as a **secret**.

Depending on the setup, examples may include:

```text
Codex   → CODEX_API_KEY or OPENAI_API_KEY
Claude  → ANTHROPIC_API_KEY
Gemini  → GEMINI_API_KEY
```

Never put the real key value in `AGENTS.md`, a task file, a workflow, or source code.

### 3. Start Claude when Codex finishes

When Codex finishes a change and opens a PR, GitHub can treat that as an event.

We can define a rule such as:

> **When a new PR opens, or new code is added to the PR, start Claude's review.**

Example workflow:

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
  add-labels:
    allowed: ["agent:fix-required", "agent:review-passed"]
---

# Review

Review the PR against AGENTS.md and the task boundaries.

Check:
- Was the requested change implemented?
- Were PRESERVE areas kept unchanged?
- Were unrelated files modified?
- Are tests or verification steps missing?

If there is a problem, report it with evidence and request the
`agent:fix-required` label.

If the change is acceptable, report the checks performed and request the
`agent:review-passed` label.
```

The two technical event names simply tell GitHub **when to run**:

- `opened` → run when the PR is first opened.
- `synchronize` → run again when a new commit is added to the same PR.

Claude then produces one of two simple states:

```text
agent:fix-required
= a correction is needed

agent:review-passed
= the review passed
```

Those states can be used to start the next step.

### 4. Send the task back to Codex when Claude finds a problem

Now we can define the second handoff:

> **If Claude produces `agent:fix-required`, start the Codex correction task.**

Example:

```markdown
---
on:
  label_command:
    name: agent:fix-required
    events: [pull_request]

engine: codex

permissions:
  contents: read
  pull-requests: read

safe-outputs:
  push-to-pull-request-branch:
---

# Fix

Review the findings on the PR.

First:
1. Record the current PR version as INPUT_VERSION.
2. Revalidate the finding against the current code.
3. Preserve the AGENTS.md rules and PRESERVE boundaries.

If the issue is confirmed, make the smallest necessary fix.
Run the relevant tests.
If the issue is not confirmed, do not change the code.

Return CHANGED, PRESERVED, VERIFIED,
SKIPPED_CHECKS, and SCOPE_STATUS.
```

We are not telling Codex **“Claude said it, so it must be true; fix it.”**

Codex first checks the current PR version and confirms that Claude's finding is still valid. Only then does it change the code if needed.

When Codex sends the correction to the PR, GitHub sees a new commit. The `synchronize` rule from the previous step runs, so Claude reviews the updated change again.

The loop can therefore continue without the user carrying messages:

```text
Claude found a problem
        ↓
Codex verified and fixed it
        ↓
PR was updated
        ↓
Claude checked again
```

### 5. Start tests when Claude passes the review

If Claude finds no problem, it can produce the `agent:review-passed` state.

The next rule can then be:

> **If the review passes, run the tests.**

```text
agent:review-passed
        ↓
test / build / lint
        ↓
pass → VERIFIED
fail → correction task
```

Not every check needs another AI agent. Unit tests, builds, and lint checks have clear results and can usually run as normal GitHub Actions steps.

AI agents are more useful when the task requires interpretation, such as reviewing whether a change stayed within scope or whether the implementation matches the requested behavior.

### 6. A trigger and an orchestrator are not the same thing

These two ideas can easily be confused.

In simple terms:

```text
Trigger
= "start now"

Orchestration
= "who should start now, and with which task?"

Agent
= "do the assigned task"
```

For example, when the `agent:fix-required` state appears in GitHub, GitHub Actions can start the Codex workflow. GitHub Actions is acting as the **trigger**.

The **workflow rule** decides which result should go to which agent.

If ChatGPT is also used as an orchestrator, there must be an integration that allows the automation to invoke ChatGPT. Without that connection, GitHub Actions and the workflow rules can manage the automated handoffs inside GitHub.

So saying **“ChatGPT is the orchestrator”** does not by itself mean that ChatGPT is automatically running inside GitHub.

### 7. Give agents only the permissions they need

Once automation is enabled, agents may act without waiting for the user. Their permissions should therefore be kept as narrow as possible.

For example, allowing an agent to work through a PR is usually more controlled than allowing it to change the `main` branch directly.

GitHub Agentic Workflows **safe outputs** can be used to limit the write actions an agent may request. For example, an agent may be allowed to create a PR, update the current PR branch, or use only specific labels.

The task can also define file boundaries:

```text
ALLOWED_FILES:
- src/navigation/**
- tests/navigation/**

PROTECTED:
- AGENTS.md
- .github/**
- dependency / package files
```

A navigation task should not automatically give an agent permission to modify unrelated workflow, security, or dependency files.

### 8. Prevent the same task from starting twice

Automation can accidentally trigger the same task more than once. If two agent runs modify the same files at the same time, they can interfere with each other.

For that reason, limit how many runs may work on the same `TASK_ID` or PR at once.

GitHub Actions **concurrency**, which controls simultaneous runs, can be used for this.

The idea is simple:

```text
UI-024 is already running
        ↓
a second UI-024 arrives
        ↓
queue / cancel / recheck
```

### 9. Start automation with one small handoff

Do not connect every agent at once in the first experiment. Start by automating one handoff that is easy to observe.

A good first target is:

```text
Codex created a PR
        ↓
Claude started reviewing automatically
```

Once that transition works reliably, add the next steps one at a time:

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

The goal is not only to run agents in sequence. **The next step should also receive the task version, the finding being transferred, the areas that must be preserved, and any checks that were not completed.**

> **Note:** GitHub Agentic Workflows is in public preview at the time of writing. During setup, check GitHub's current engine, trigger, permission, and safe-output options.

## When to use it

This method is especially useful for limited fixes to existing files or code, when working parts must be preserved, or when an agent should touch only specific files.

A detailed change-boundary record may not be necessary for small experiments that are easy to discard. For example, when comparing a few text formats in an empty test file, defining protected lines is usually unnecessary.
