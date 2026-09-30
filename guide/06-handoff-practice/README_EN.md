# Handoff Practice

[Türkçe](README.md)

> This section shows how to apply the rules from [Agent Handoffs](../03-agent-handoffs/README_EN.md) in a workflow using **ChatGPT, Codex, Claude, Gemini, and GitHub**.

The goal here is not to build automation. First, we make the handoff logic visible through a controlled manual flow. Agents automatically starting one another will be covered in **Agent Automation**.

## How is this different from the earlier handoff section?

**Agent Handoffs** explains what information must not be lost during a handoff.

**Handoff Practice** shows **who passes that information to whom, in what order, and with which checks** in an actual multi-agent workflow.

## Working structure

This section uses the following roles:

| Part | Role in this example |
|---|---|
| **ChatGPT** | Structures the task, collects results, and prepares the next step. |
| **Gemini** | Performs testing, analysis, or a second-opinion review. |
| **Codex** | Makes the required code or file change. |
| **Claude** | Independently reviews the completed change. |
| **GitHub** | Shared workspace for files, recorded changes, and review records. |
| **Human** | Approves scope changes and the final decision to add changes to the main project. |

The roles are fixed here to keep the example easy to follow. The core rule is that one agent's result is **not accepted as correct by the next agent without verification**.

## Big picture

**EXAMPLE — Read only**

```text
Human
  |
  v
ChatGPT
Prepares the task and boundaries
  |
  v
Gemini
Tests / analyzes
  |
  v
ChatGPT
Checks the result and current version
  |
  v
Codex
Verifies the finding and makes the smallest change if needed
  |
  v
Claude
Independently reviews the change
  |
  v
Human
Makes the final decision

GitHub = shared workspace for files, versions, and change records
```

In this section, one agent finishing does not automatically start the next agent. The user or ChatGPT passes the controlled task package to the next agent. Automatic triggering will be covered later.

## Core rule

> **A finding from one agent is not an instruction for the next agent; it is information that must be rechecked against the current state.**

For example, if Gemini finds a problem, Codex is not simply told “fix this.”

Instead, the handoff carries:

```text
Gemini observed this problem in this version.
Here is the evidence.
Here are the checks Gemini could not perform.
Verify whether the finding still applies to the current version.
If verified, make the smallest change within the task boundaries.
```

## Task package

The same basic structure can be used at every agent transition.

**EXAMPLE — Read only**

```text
TASK_ID:          [Short task identifier]
INPUT_VERSION:    [Version that was inspected]

TASK:             [Work to perform]
FILES / CONTEXT:  [Files and information used]

CHANGE:           [What may change]
PRESERVE:         [What must remain unchanged]
DO NOT:           [What must not be done]
LIMITATIONS:      [What the agent could not see or check]

EXPECTED_OUTPUT:  [Expected result format]
RETURN_TO:        [Who receives the result first?]
NEXT_AGENT:       [Who receives the task after review?]
ROUND:            [Which round? Example: 1/3]
```

Two fields serve different purposes:

- **RETURN_TO:** Who receives the result first when the agent finishes?
- **NEXT_AGENT:** Which agent takes over after that result has been checked?

For example, Gemini's result can return to ChatGPT first; after ChatGPT checks it, the task can move to Codex.

## Why does the version travel with the task?

While one agent is inspecting a file, another agent may make a new change to the same file. An older finding may no longer be valid.

The task package therefore carries **INPUT_VERSION (the inspected version)**. The current version on GitHub is later recorded as **CURRENT_VERSION (the version now)**.

Each recorded change in Git has an identifier called a **commit ID**.

**EXAMPLE — Read only**

```text
INPUT_VERSION:    3f9a2c1
CURRENT_VERSION:  3f9a2c1
VERSION_STATUS:   MATCH
                  -> Same version. Review can continue.

INPUT_VERSION:    3f9a2c1
CURRENT_VERSION:  8b41d07
VERSION_STATUS:   CHANGED
                  -> The version changed. Do not apply the old finding
                     directly. Reverify it against the current version.
```

Rule:

> **If the version changed, do not apply the old finding directly; verify it again against the current state first.**

## Example workflow: Gemini → ChatGPT → Codex → Claude

The following example shows a test finding moving through implementation and independent review.

### 1. ChatGPT prepares the task

The human gives the work to ChatGPT. ChatGPT turns it into a small, explicit task package.

**PASTE TO CHATGPT**

```text
Turn this work into a multi-agent task package.

TASK:
[Write the work here.]

First define:
CHANGE
PRESERVE
DO NOT

Give the task a short TASK_ID.
Record the version to be used as INPUT_VERSION.
Gemini will perform the first check.
The result will return to ChatGPT first.
If a change is needed, the next implementation agent will be Codex.

If a boundary is missing or ambiguous, do not expand the task.
Ask me before changing the scope.
```

### 2. Gemini reviews

Give Gemini the task package and the files required for the task. Gemini does not modify files at this stage; it produces observations and evidence.

**PASTE TO GEMINI**

```text
Review the task package below.

Use only the files and information you can actually access.
Do not describe an unseen area as checked.

Return:

TASK_ID:
INPUT_VERSION:
OBSERVED: [What you directly observed]
INTERPRETED: [Your interpretation of those observations]
FINDINGS: [Problems you found]
EVIDENCE: [File, line, or information supporting each finding]
VERIFIED: [What you actually checked]
SKIPPED_CHECKS: [Checks you could not perform]
RECOMMENDED_ACTION:
RETURN_TO: ChatGPT
NEXT_AGENT: Codex

Task package:
[Add the task package prepared by ChatGPT here.]
```

Keep **OBSERVED** separate from **INTERPRETED** so the next agent does not mistake an interpretation for evidence.

### 3. ChatGPT performs the transition check

Gemini's result is not sent directly to Codex. ChatGPT first checks the task identity, version, evidence, and skipped checks.

**PASTE TO CHATGPT**

```text
Check the Gemini result below as a handoff.

1. Is the TASK_ID correct?
2. Which version does INPUT_VERSION refer to?
3. Determine the current GitHub version and record it as CURRENT_VERSION.
4. If INPUT_VERSION and CURRENT_VERSION differ, do not pass the finding
   as a direct fix task; state that reverification is required.
5. If they match, prepare a Codex task package without losing FINDINGS,
   EVIDENCE, SKIPPED_CHECKS, CHANGE, PRESERVE, or DO NOT.
6. Do not expand the scope automatically.
7. Do not change code.

Gemini result:
[Add Gemini's result here.]
```

ChatGPT's job here is not to repeat the test. It checks whether **the handoff information is complete and still current**.

### 4. Codex verifies the finding and changes only when needed

Codex does not begin by assuming Gemini's interpretation is correct. It first verifies the finding against the current files.

**PASTE TO CODEX**

```text
Apply the task package below.

First compare the current version you can access with CURRENT_VERSION
in the package. If the version changed, do not modify code; report that
the finding needs to be reverified.

If the version matches:
1. Verify the finding yourself against the current code and tests.
2. If it is not verified, do not change anything; report the evidence.
3. If it is verified, make the smallest required change within
   CHANGE / PRESERVE / DO NOT.
4. Run the relevant checks or tests.
5. If the task requires expanding the scope, stop and request human approval.

Return:
TASK_ID:
CURRENT_VERSION:
CHANGED:
PRESERVED:
VERIFIED:
SKIPPED_CHECKS:
UNEXPECTED_CHANGES:
SCOPE_STATUS:

Task package:
[Add the package prepared by ChatGPT here.]
```

The change-boundary rules are covered in [Safe Changes](../02-safe-changes/README_EN.md).

### 5. Claude performs an independent review

Codex reporting “done” is not the final check. Claude independently compares the original task boundaries with the actual changes.

**PASTE TO CLAUDE**

```text
Independently review the task below.

Compare the original TASK / CHANGE / PRESERVE / DO NOT information,
Codex's result, and the actual changes recorded on GitHub.

Check:
- Was the requested change made?
- Were PRESERVE areas kept unchanged?
- Was any DO NOT boundary crossed?
- Did an unexpected file or behavior change?
- Is there real evidence for what Codex marked VERIFIED?
- Are skipped checks clearly recorded?

Return:

TASK_ID:
REVIEWED_VERSION:
OBSERVED:
FINDINGS:
EVIDENCE:
SKIPPED_CHECKS:
SCOPE_STATUS:
RECOMMENDATION: READY_FOR_HUMAN / NEEDS_CHANGES / NEEDS_RECHECK
```

Claude's result is not the final decision either. Final acceptance belongs to the human.

## When does the human decide?

The flow does not continue automatically in these cases:

| Situation | Human decision |
|---|---|
| The scope needs to expand | Approve the new boundary or narrow the task. |
| The inspected version and current version differ | Request a new check or stop the task. |
| Agents disagree about the same finding | Compare evidence and request another check if needed. |
| There is an unexpected change | Investigate why; revert it if it is not required. |
| The round limit is reached | Split, redefine, or stop the task. |
| The change is ready to be added to the main project | Review the final change difference and make the **merge** decision. |

## Limit the loop

Review → fix → review can otherwise continue indefinitely.

The task package can include:

```text
ROUND: 1/3
```

Increase the number on each new round. At the final round, agents do not start another correction cycle on their own; the task returns to the human.

The number `3` is not a universal rule. The purpose is to set an explicit boundary that prevents uncontrolled repetition.

## A written boundary is not a technical lock

CHANGE, PRESERVE, and DO NOT tell the agent how it should behave; they do not technically prevent the agent from crossing the boundary.

Therefore:

1. Compare the agent's result with the original task boundary.
2. Inspect the actual change difference (**diff**) on GitHub.
3. Do not expand critical scope or authority without human approval.
4. Keep the final **merge** decision with the human.

## Where do shared rules live?

Persistent project rules can be kept in a shared project instruction such as `AGENTS.md`.

However, a file existing on GitHub does not mean ChatGPT, Codex, Claude, and Gemini all read it automatically.

If an agent cannot access the shared instruction file, carry the rules required for that task explicitly through **PRESERVE / DO NOT / LIMITATIONS** in the task package.

Shared instruction loading and the handoff record structure are covered in [Agent Handoffs](../03-agent-handoffs/README_EN.md).

## Summary

1. ChatGPT structures the task and its boundaries.
2. Gemini produces observations, evidence, and skipped checks.
3. ChatGPT checks the version and handoff information.
4. Codex reverifies the finding and makes the smallest change when needed.
5. Claude independently reviews the change.
6. GitHub holds the shared files, versions, and change records.
7. Scope expansion and the final merge decision remain with the human.
8. Task identity, version, scope, evidence, and skipped checks travel with every handoff.

## Next step

- **Agent Automation** — How this controlled handoff can run through triggers and automatic agent transitions without manually carrying messages. This section has not been published yet.
