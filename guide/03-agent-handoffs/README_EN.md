# Agent Handoffs

[🇹🇷 Türkçe](README.md)

## Problem

One agent may complete its task correctly, but the next agent can still make a mistake if it does not receive the information needed to continue the work.

For example, Agent A may know that an important part of a file must remain unchanged. If that requirement is not passed to Agent B, Agent B may accidentally undo or damage the previous work.

## Core rule

> **A handoff should clearly state what changed, what must be preserved, which checks were completed, and which checks are still pending.**

## Handoff Receipt

A handoff can include:

- **TASK_ID**
- **INPUT_VERSION**
- **OUTPUT_VERSION**
- **CHANGED**
- **PRESERVE**
- **VERIFIED**
- **SKIPPED_CHECKS**
- **NEXT_AGENT**

## Record what was actually checked

A statement such as “I checked everything” is not enough by itself. When the task has a known scope, record how many items were expected and how many were actually checked.

Useful fields include EXPECTED_ITEMS, OBSERVED_ITEMS, CHECKED_ITEMS, SKIPPED_ITEMS, and COVERAGE_STATUS.

For example, if a task contains 12 files but only 10 were reviewed, the remaining 2 should be clearly recorded as unchecked.

## Example

Agent A changes three fictional configuration files but can test only two because a required runtime is unavailable.

The handoff records:

- 3 files were changed.
- 2 files were tested.
- 1 file could not be tested.
- The reason the test could not be run is recorded.

Agent B can immediately see what is complete and what still needs verification.

## How to apply it

Add a short handoff request to the instructions used when one agent passes work to another.

For example:

> **At the end of the task, create a handoff record. State the input and output versions, what you changed, what must remain unchanged, which checks you completed, which checks you could not complete, and what the next agent should do. Do not describe an unchecked item as verified.**

If the task has a fixed number of files, records, tests, or other items, also ask the agent to report the expected, checked, and skipped counts.

In automated multi-agent workflows, the same fields can be stored as structured data and passed directly to the next agent instead of relying only on free-form text.

## When to use it

A handoff record is useful when work moves between agents, sessions, machines, or people.

For a small one-step task that will not be passed to anyone else, a detailed handoff record may not be necessary.
