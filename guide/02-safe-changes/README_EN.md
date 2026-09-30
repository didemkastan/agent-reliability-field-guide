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

NEXT_AGENT:
NEXT_ACTION:
```

The package states which version the agent is working from, which files or information it can see, what it should test, what must be preserved, and what it cannot access. The external agent returns its result with the same `TASK_ID` and `INPUT_VERSION`.

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

## Use in automation

In an automated workflow, expected files can be defined before the task starts. After the task, the system can compare that list with the files that actually changed.

If an unexpected file was modified, the change can be held for review instead of being accepted automatically. Stricter workflows can also compare changed lines with the areas that were allowed to change.

## When to use it

This method is especially useful for limited fixes to existing files or code, when working parts must be preserved, or when an agent should touch only specific files.

A detailed change-boundary record may not be necessary for small experiments that are easy to discard. For example, when comparing a few text formats in an empty test file, defining protected lines is usually unnecessary.
