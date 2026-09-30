[🇹🇷 Türkçe](handoff-receipt.md)

# Handoff Receipt Template

This template provides a shared record structure for task handoffs.

> **This template is not an official standard from any particular product or platform. It combines the principles used in this guide to help preserve version, change, verification, coverage, and next-step information during agent handoffs.**

## Who fills it out?

The agent or person handing off the task completes the record. It is not only a summary of completed work; it should clearly show where the next party needs to continue.

## When is it used?

It can be used when a task moves to another agent or person, continues in a different session, or requires another party to complete the remaining checks.

The party handing off the task first completes the checks it can perform and then creates the handoff receipt.

## Where is it stored?

The location depends on the working environment:

- For a one-off task, it can be provided directly in the chat.
- In a shared project workspace, it can be stored in a file such as `HANDOFF.md`.
- In an automated agent workflow, the same fields can be transferred as JSON or a task record.

The existence of a handoff receipt does not automatically start the next agent. Automated transitions require a separate trigger and orchestration mechanism.

## Ready-to-use template

```text
HANDOFF RECEIPT

TASK_ID:
INPUT_VERSION:
OUTPUT_VERSION:

CHANGED:
PRESERVE:

VERIFIED:
SKIPPED_CHECKS:

EXPECTED_ITEMS:
OBSERVED_ITEMS:
CHECKED_ITEMS:
SKIPPED_ITEMS:
COVERAGE_STATUS:

NEXT_AGENT:
NEXT_ACTION:
```

## Filled example

Below is a simple example showing how the template can be completed:

```text
HANDOFF RECEIPT

TASK_ID: DOC-014
INPUT_VERSION: v1.2
OUTPUT_VERSION: v1.3

CHANGED:
- Corrected the installation command in README.md.
- Added the missing configuration example.

PRESERVE:
- Do not change the existing folder structure.
- Keep the current command names unchanged.

VERIFIED:
- README.md was reopened and the changes were confirmed.
- The installation command was run successfully.

SKIPPED_CHECKS:
- Windows environment was not tested.

EXPECTED_ITEMS: 3
OBSERVED_ITEMS: 3
CHECKED_ITEMS: 2
SKIPPED_ITEMS: 1
COVERAGE_STATUS: 2/3 checked

NEXT_AGENT: Agent B
NEXT_ACTION: Run the installation command in a Windows environment and record the result.
```

This example makes both completed work and the remaining check visible. The receiving agent can see where to continue without treating an unperformed check as verified.

## What do the fields mean?

- **TASK_ID:** Identifier of the task being transferred.
- **INPUT_VERSION:** Version used when the work started.
- **OUTPUT_VERSION:** Version produced when the work ended.
- **CHANGED:** Changes made during the task.
- **PRESERVE:** Elements or constraints the next agent must preserve.
- **VERIFIED:** Results that were actually checked and verified.
- **SKIPPED_CHECKS:** Checks that could not be performed or were intentionally skipped.
- **EXPECTED_ITEMS:** Total items expected to be examined.
- **OBSERVED_ITEMS:** Items actually observed during the work.
- **CHECKED_ITEMS:** Items actually checked.
- **SKIPPED_ITEMS:** Items that were not checked.
- **COVERAGE_STATUS:** Indicates how much of the expected scope was checked.
- **NEXT_AGENT:** Agent or party expected to take over the task.
- **NEXT_ACTION:** First action the receiving party should perform.

## What does the next agent do?

The receiving agent reads the record but does not treat its contents as verified facts without checking them. Before continuing, it confirms that the relevant files and versions are still current, respects the constraints in **PRESERVE**, and continues especially from **SKIPPED_CHECKS** and **NEXT_ACTION**.

> **A handoff receipt makes the work visible; it does not replace verification.**

## Privacy

Public receipts should not contain company information, customer data, credentials, access keys, private code, confidential logs, or other sensitive information.
