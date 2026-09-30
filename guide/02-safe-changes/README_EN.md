# Safe Changes

[Türkçe](README.md)

> This section teaches one thing: **in a multi-agent workflow, how do we make sure only the requested area changes while protected areas remain intact?**

This guide uses a working structure built around **ChatGPT, Codex, Claude, Gemini, and GitHub**. Handoffs and automation are explained separately in later sections.

## Problem

When an agent receives a small change request, it may modify areas outside the task while producing the requested result.

This matters more in a multi-agent workflow. An unnecessary change made by the first agent can be passed to the next agent, which may continue while treating that change as correct.

As a result, unwanted changes can travel through the agent chain even when the original task was completed correctly.

## Core rule

> **Before a change begins, clearly define what may change, what must be preserved, and what must not be done. Make the smallest change required for the requested result.**

This boundary should remain intact from the agent making the change to the agent reviewing or continuing the task.

## Change boundary: three fields

- **CHANGE:** The requested change.
- **PRESERVE:** Areas that must remain unchanged.
- **DO NOT:** Actions that must not be performed during this task.

**EXAMPLE — Read only**

```text
CHANGE:
- Replace the old link in README.md with the new one.

PRESERVE:
- Other text
- Heading order
- File structure

DO NOT:
- Change other files.
- Rewrite unrelated text.
```

The list does not need to be long. What matters is that the task boundary is visible before the change begins.

## Smallest necessary change

If a typo can be fixed on one line, there is no need to rewrite the paragraph or modify other files at the same time.

If an agent notices another problem outside the scope, it should **report it** rather than fix it automatically. If the scope needs to expand, define the new boundary explicitly first.

## Expected and actual changes

If the requested change is only to update one link in `README.md`:

**EXAMPLE — Read only**

```text
EXPECTED                ACTUAL
README.md               README.md
                        config.yaml     <- unexpected
                        package.json    <- unexpected
```

Even if the link was fixed correctly, the other two files were not part of the original task.

Ask both questions:

1. **Was the requested result produced?**
2. **Were only the allowed changes made?**

## How is scope checked?

The first scope check can be performed by an agent other than the one that made the change. For example, if Codex made the change, Claude or Gemini can inspect the diff. However, before a change is merged into the main branch, the final scope review and acceptance decision belongs to the human.

Compare CHANGE with the actual change and answer:

- Did an unexpected file change?
- Was an area listed under PRESERVE modified?
- Was an action prohibited under DO NOT performed?
- Was any additional change made that was not required for the task?

The agent's own report is not verification by itself. Inspect the actual **diff**.

### Check the diff on GitHub

If the change is in a Pull Request (PR) on GitHub, open the PR and select **Files changed**. This shows the changed files and lines.

An independent agent can review the change here, and the human reviews the same diff before the final merge decision.

### Check the diff on your computer

Run these commands in Terminal or PowerShell from the repository folder.

**RUN IN TERMINAL**

```bash
git status
git diff --stat
git diff
```

- `git status` shows which files changed.
- `git diff --stat` summarizes changes by file.
- `git diff` shows the changed lines.

> A normal `git diff` here is used to inspect working-tree changes that have not yet been committed. If the agent already committed the change, use GitHub's **Files changed** view or inspect the latest commit separately with `git show HEAD`.

## What happens when an unexpected change is found?

Do not accept an unexpected change automatically. First determine why it exists.

### Option 1 — Ask the agent to explain and revert it

**PASTE TO THE AGENT**

```text
Compare the current diff with the original CHANGE / PRESERVE / DO NOT boundaries.
List every out-of-scope change.
Explain why each unexpected change occurred.
If it is not required to complete the task, revert only the out-of-scope change.
Do not expand the scope automatically.
```

Check the diff again after the correction.

### Option 2 — Revert an uncommitted local file

First inspect the file:

**RUN IN TERMINAL**

```bash
git diff config.yaml
```

If you are **certain you want to discard all uncommitted local changes in that file**:

**RUN IN TERMINAL**

```bash
git restore config.yaml
```

> **Warning:** `git restore config.yaml` can discard uncommitted changes in that file. Do not run it if you are unsure. If the change was already committed, do not use this command as the solution; inspect the commit diff first.

## How is scope preserved between agents?

CHANGE / PRESERVE / DO NOT should not remain only in the first task message. When work moves from one agent to another, the scope and actual result move together.

```text
TASK
  |
  +-- CHANGE
  +-- PRESERVE
  +-- DO NOT
  |
  v
Agent work
  |
  v
CHANGED + UNEXPECTED_CHANGES + SCOPE_STATUS
  |
  v
Independent scope check
  |
  v
Next agent receives task + scope + result
```

At minimum, preserve these fields across each transition:

```text
CHANGE
PRESERVE
DO NOT
CHANGED
UNEXPECTED_CHANGES
SCOPE_STATUS
```

How persistent shared rules are stored in project instructions such as `AGENTS.md` and carried to the next agent is covered in [Agent Handoffs](../03-agent-handoffs/README_EN.md).

## Ready-to-use task instruction

**PASTE TO THE AGENT — Replace the bracketed placeholders with your task information**

```text
Check the task scope before starting.

CHANGE:
- [Write the file, section, or behavior you want changed here.]

PRESERVE:
- [Write the files, sections, or working behavior that must remain unchanged here.]

DO NOT:
- [Write the actions that must not be performed in this task here.]

Make the smallest change required to produce the requested result.

If you notice another problem outside the scope, do not fix it automatically.
Report it separately.

If completing the task requires expanding the scope, stop before making
that change and explain why.

When finished, compare the actual changes with the original scope.

Return:
CHANGED: [Write what actually changed.]
PRESERVED: [Write what you checked remained unchanged.]
UNEXPECTED_CHANGES: [Write unexpected changes; if none, write none.]
SCOPE_STATUS: [IN_SCOPE or OUT_OF_SCOPE]
```

## When is it used?

This rule is especially important when an agent makes a project change in the multi-agent workflow and:

- A limited change is made to working code or files.
- Specific areas must remain unchanged.
- One agent's change is handed to another agent.
- The same task is reviewed or verified by another agent.

### When is the detailed version unnecessary?

A detailed CHANGE / PRESERVE / DO NOT record may be unnecessary for small experiments that are easy to discard and do not affect real project behavior. For example, comparing a few text formats in an empty test file usually does not require a full scope record.

## Summary

1. Define **CHANGE / PRESERVE / DO NOT** before the change begins.
2. The agent makes only the **smallest necessary change**.
3. Inspect the actual change through the **diff**.
4. Investigate unexpected changes and revert them when they are not required.
5. Carry scope information to the next agent together with the task.
6. Independent agent review can help; **the human makes the final acceptance decision before merge.**

## Next steps

- [**Agent Handoffs**](../03-agent-handoffs/README_EN.md) — How task scope, version information, and results move between agents.
- **Agent Automation** — How safe handoffs are implemented with automated triggers and controlled agent transitions. This section has not been published yet.
