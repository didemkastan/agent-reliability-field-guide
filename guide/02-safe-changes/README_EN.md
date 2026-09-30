# Safe Changes

[Türkçe](README.md)

> This section teaches one thing: **in a multi-agent workflow, how do we make sure only the requested area changes while protected areas remain intact?**

This guide uses a working structure built around **ChatGPT, Codex, Claude, Gemini, and GitHub**. How tasks move between agents and how this structure is automated will be explained in later sections.

## Problem

When an agent receives a small change request, it may modify areas outside the task while producing the requested result.

This becomes more important in a multi-agent workflow. An unnecessary change made by the first agent can be passed to the next agent, which may continue working while treating that change as correct.

As a result, even when the original task is completed correctly, unwanted changes can travel through the agent chain.

## Core rule

> **Before a change begins, clearly define what may change, what must be preserved, and what must not be done. Make the smallest change required for the requested result.**

This boundary applies not only to the agent making the change, but also to agents that later review or continue the task.

## Change boundary: three fields

- **CHANGE:** The requested change.
- **PRESERVE:** Areas that must remain unchanged.
- **DO NOT:** Actions that must not be performed during this task.

**Example task instruction**

```
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

If an agent notices another problem outside the scope, it should **report it** rather than fix it automatically.

If the scope needs to expand, that new need is made visible first. This prevents the next agent from confusing the original task with a newly discovered issue.

## Expected and actual changes

If the requested change is only to update one link in `README.md`, the expected scope is:

```
EXPECTED                ACTUAL
README.md               README.md
                        config.yaml     <- unexpected
                        package.json    <- unexpected
```

Even if the link was fixed correctly, the other two files were not part of the original task.

So it is not enough to ask only:

**“Was the requested result produced?”**

A second question is also required:

> **“Were only the allowed changes made?”**

## How is scope checked?

When the work is complete, compare two things:

1. **Expected changes:** What was defined in CHANGE before work started.
2. **Actual changes:** The files and areas the agent actually modified.

Check whether:

- An unexpected file changed.
- An area listed under PRESERVE was modified.
- An action prohibited under DO NOT was performed.
- An additional change was made that was not required to complete the task.

Do not rely only on the agent's own report. Whenever possible, inspect the actual change difference (**diff**).

## Scope record in a multi-agent workflow

CHANGE / PRESERVE / DO NOT should not remain only in the first task message.

When work moves from one agent to another, the same scope information moves with it.

```
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
Actual change
  |
  v
Scope check
  |
  v
Next agent receives task + scope + result
```

This means the next agent sees not only the resulting files, but also **which changes were allowed and which areas had to remain protected**.

How the handoff is recorded and how version information travels with the task will be covered in the next section.

## Ready-to-use task instruction

```
Check the task scope before starting.

CHANGE:
Make only the changes listed here.

PRESERVE:
Keep the files, sections, and working behavior listed here unchanged.

DO NOT:
Do not touch the areas or perform the actions prohibited here.

Make the smallest change required to produce the requested result.

If you notice another problem outside the scope, do not fix it automatically.
Report it separately.

If completing the task requires expanding the scope, stop before making
that change and explain why.

When finished, compare the actual changes with the original scope.

Return:
CHANGED: What actually changed
PRESERVED: What you checked remained unchanged
UNEXPECTED_CHANGES: Unexpected changes (or: none)
SCOPE_STATUS: Did the change stay within the task boundary?
```

> **An agent saying “I stayed within scope” is not verification by itself.** The actual changes should be checked separately whenever possible.

## How is this rule preserved between agents?

In the structure used by this guide, ChatGPT, Codex, Claude, and Gemini can work at different stages of the same task. GitHub can serve as the shared workspace.

The important point in this section is not which agent owns which role, but that **the task boundary must not disappear when the agent changes**.

At minimum, preserve these fields across each transition:

```
CHANGE
PRESERVE
DO NOT
CHANGED
UNEXPECTED_CHANGES
SCOPE_STATUS
```

The next agent does not automatically accept the previous agent's change as correct. It first compares the task boundary with the actual change.

The record structure used for this transfer will be covered in **Handoff Practice**.

## When is it used?

This rule can be used whenever an agent makes a project change inside the multi-agent workflow.

It is especially important when:

- A limited change is made to working code or files.
- Specific areas must remain unchanged.
- One agent's change is handed to another agent.
- The same task is reviewed or verified by multiple agents.

## Summary

1. Define **CHANGE / PRESERVE / DO NOT** before the change begins.
2. The agent makes only the **smallest necessary change**.
3. Compare the actual change with the original scope.
4. Do not automatically treat unexpected changes as part of the task.
5. Carry the scope information to the next agent together with the task.
6. The agent's own report is not verification by itself.

## Next steps

- **Handoff Practice:** How the task, scope, version information, and results move between agents.
- **Agent Automation:** How safe handoffs are implemented with automated triggers and controlled agent transitions.
