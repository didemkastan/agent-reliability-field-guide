# Agent Handoffs

[🇹🇷 Türkçe](README.md)

## Problem

One agent may complete its task correctly, but the next agent can still make a mistake if it does not receive the information needed to continue the work.

For example, Agent A may know that an important part of a file must remain unchanged. If that requirement is not passed to Agent B, Agent B may accidentally undo or damage the previous work.

A “handoff” does not necessarily mean that agents communicate directly with each other. In many setups there is no direct agent-to-agent communication channel. The handoff information may be carried by the user, written to a shared file, or transferred to the next agent by an automation.

## Core rule

> **A handoff should clearly state what changed, what must be preserved, which checks were completed, which checks are still pending, and what should happen next.**

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
- **NEXT_ACTION**

## Record what was actually checked

A statement such as “I checked everything” is not enough by itself. When the task has a known scope, record how many items were expected and how many were actually checked.

Useful fields include EXPECTED_ITEMS, OBSERVED_ITEMS, CHECKED_ITEMS, SKIPPED_ITEMS, and COVERAGE_STATUS.

For example, if a task contains 12 files but only 10 were reviewed, the remaining 2 should be clearly recorded as unchecked.

## Example

Agent A changes three example configuration files but can test only two because a required runtime is unavailable.

The handoff records:

- 3 files were changed.
- 2 files were tested.
- 1 file could not be tested.
- The reason the test could not be run was recorded.
- The check that the next agent should complete was identified.

The next agent can see what is complete and where work should continue.

## How to apply it

A handoff can be implemented in three ways. The right approach depends on whether the agents share project files and whether an automated transfer mechanism exists between them.

### 1. User-mediated handoff

If two separate agents such as ChatGPT and Claude do not have a direct communication channel, the user starts the handoff.

At the beginning of the task, or while it is in progress, the first agent can be given this instruction:

> **Work so that another agent can take over this task. At the end, create a Handoff Receipt. Include the input and output versions, what you changed, what must be preserved, completed and skipped checks, verification results, and the first action the next agent should take. Do not describe an unchecked item as verified.**

When the first agent finishes, the user gives this record to the second agent. Instead of saying only “continue,” the user can provide the handoff record with this instruction:

> **Review this Handoff Receipt. Before making changes, confirm that the referenced files and versions are still current. Do not change the rules listed under PRESERVE. Continue from SKIPPED_CHECKS and NEXT_ACTION. Do not accept the information in the receipt as correct without checking it.**

In this setup, **the user acts as the bridge**. The user does not need to rewrite the technical details; transferring the handoff record created by the first agent is enough.

### 2. Shared project file

When several agents regularly work on the same project, the handoff rule does not have to be typed again for every task.

The rule can be stored in a persistent instruction file that the agent is configured to read. Depending on the tool, this may be an `AGENTS.md`, a project instruction file, or a similar mechanism. A normal `README.md` can document the project for people, but you should not assume that every agent automatically treats the README as task instructions.

A separate file such as `HANDOFF.md` can hold the current handoff record. Agent A updates it when finishing the task. The next agent reads it at startup and verifies that the recorded state is still current.

Example flow:

**User assigns task to Agent A → Agent A works → Agent A creates the handoff record → User or shared workspace makes the record available to Agent B → Agent B rechecks versions and conditions → Agent B continues the work.**

### 3. Automated multi-agent system

If an orchestration system or another automation can transfer data between agents, the user does not need to manually carry the record at every handoff.

The Handoff Receipt fields can be stored as structured data. The system adds Agent A's output, the handoff record, and the required file/version information to Agent B's input. Agent B still verifies the current state before starting work.

Automation **moves the information**; it does not remove the need for verification.

## A handoff does not automatically start the next agent

A shared workspace such as GitHub can give agents access to the same files and task records. However, Agent A creating a file or handoff record does not mean that Agent B will automatically start working.

There are three separate parts:

- **Handoff:** Defines what information is passed to the next agent.
- **Trigger:** Determines when the next agent is started.
- **Orchestrator:** Manages which agent is next and what task information it receives.

Without automation, the user starts the first agent, reviews the handoff record, and starts the next agent with that record. With automation, an event such as a task status change or pull request in GitHub can be used to trigger the next step.

**In short:** The shared workspace carries the information; the trigger starts the agent; the handoff record tells the agent what it needs to continue.

## When does the human act?

Without direct agent-to-agent transfer, the user typically:

1. Chooses which agent starts the task and assigns it.
2. Requests a handoff record when the work needs to move to another agent.
3. Gives the handoff record and, when necessary, the relevant files to the next agent.
4. Tells the next agent to verify the record and continue.
5. Separately approves any change in authority, scope, or important decisions.

The user does not need to repeat the technical work performed by the agents. But without an automated transfer mechanism, **the user initiates the handoff and delivers the record to the next agent.**

## When to use it

Use this method when work will move from one agent to another agent or person, when work will continue in another session, when different tools will work on the same project in sequence, or when someone else will need to continue the task later.

It is especially useful for file or code changes, when testing will be completed by another agent, and when long-running work continues across different sessions.

A detailed handoff record may not be necessary for a small, reversible task completed by one agent in a single session with no later transfer.
