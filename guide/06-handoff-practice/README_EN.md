# 6. Handoff Practice (Agent Handoff in Practice)

[🇹🇷 Türkçe](README.md)

## Purpose
Help one AI agent safely hand over ongoing work to another without losing information or repeating unnecessary steps.

## 1. Problem
When an agent changes code, the next agent may not know which files changed, what must be preserved, or which tests were run. This can cause errors, duplicated work, and regressions in working code.

## 2. Core Rule
**Before starting, the receiving agent must understand the current state and the boundaries of the task.**

Each handoff should include five pieces of information:
- **Task:** What needs to be done?
- **Current state:** What has already been done?
- **Preserve:** What must not change?
- **Verification:** Which checks were completed, and which are still missing?
- **Next action:** What should the next agent do?

## 3. Example: ChatGPT → Codex → Reviewer
**Scenario:** An application has a report-generation bug.

**ChatGPT — Defines the task**

> Task: Fix the report-generation bug.
>
> Current state: The issue has been reproduced and the relevant file identified.
>
> Preserve: Keep the existing screen design and other report types unchanged.
>
> Verification: The failing scenario is known, but the fix has not been tested yet.
>
> Next action: Fix only this issue and report the test results.

**Codex — Makes the change.** It records the changes, test results, and skipped checks, then hands the task to the reviewer.

**Reviewer — Checks the result.** The reviewer inspects the changes, independently runs tests when possible, and reports:
- `PASS`: Required checks passed.
- `FAIL`: An error or failing test was found.
- `NEEDS_REVIEW`: There is not enough evidence to verify the result.

## 4. How to Apply It
1. Define the task and what must be preserved.
2. Check the current file and version.
3. Hand over the task using the [Handoff Receipt](../../patterns/handoff-receipt_EN.md) template.
4. Confirm that the receiving agent accepted the task.
5. Check the test evidence when the work is complete.

**Important:** An agent saying “done” is not verification by itself.

## 5. When to Use It
Use this when multiple AI agents work on the same development task or when another agent will continue the work. A detailed handoff is not necessary for a small task completed by one agent.

## Conclusion
**A good handoff communicates not only what needs to be done, but also what has been done, what must be preserved, and what has actually been verified.**

This method can be used manually. The next section will explain how to automate the same process with GitHub.
