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

### Example: task flow with ChatGPT, Codex, Claude, and GitHub

Suppose ChatGPT, Codex, and Claude are used together on the same software project. GitHub is not an agent in this example; it is the shared workspace where project files, versions, and change history are maintained.

A simple division of work could look like this:

```text
GitHub
└── Canonical project files + AGENTS.md
              ↓
ChatGPT
└── Analyzes the request and prepares the task boundary.
    TASK / CHANGE / PRESERVE / DO NOT
              ↓
Codex
└── Receives the rules and task boundary.
    Makes the required code change.
    Produces CHANGED / PRESERVED / SCOPE_STATUS.
              ↓
GitHub
└── The change and current version are recorded in the shared workspace.
              ↓
Claude
└── Receives the current version and handoff record.
    Reviews the change and the areas that should have been preserved.
    Reports missing or unexpected changes.
              ↓
GitHub
└── The verified result and current project state are preserved.
```

In this example, ChatGPT handles task preparation and routing, Codex performs the implementation, and Claude provides a second review. These roles do not have to be fixed; they can change according to the task. What matters is that every agent receives the same canonical rules and the current task state.

For this flow to run automatically, an **orchestrator or triggering mechanism** is also required. Giving multiple agents access to the same GitHub repository does not, by itself, cause one agent to start automatically when another finishes.

## Use in automation

In an automated workflow, expected files can be defined before the task starts. After the task, the system can compare that list with the files that actually changed.

If an unexpected file was modified, the change can be held for review instead of being accepted automatically. Stricter workflows can also compare changed lines with the areas that were allowed to change.

## When to use it

This method is especially useful for limited fixes to existing files or code, when working parts must be preserved, or when an agent should touch only specific files.

A detailed change-boundary record may not be necessary for small experiments that are easy to discard. For example, when comparing a few text formats in an empty test file, defining protected lines is usually unnecessary.
