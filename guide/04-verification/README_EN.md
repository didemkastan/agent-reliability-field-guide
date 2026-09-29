# Verification

[🇹🇷 Türkçe](README.md)

## Why verification matters

An agent reporting “done” does not by itself show that the expected result actually occurred. The result should be checked after the action is complete.

Keep these four pieces of information separate:

- **OBSERVED** — Information directly seen in a file, test, tool, or system result.
- **INTERPRETED** — An assessment of what the observation means.
- **ACTION** — The change or operation that was performed.
- **VERIFIED** — The check showing whether the expected result occurred after the action.

## Example

An agent changes a configuration file to enable a feature.

- **OBSERVED:** The file contains `feature_enabled: false`.
- **INTERPRETED:** This value indicates that the feature is disabled.
- **ACTION:** The value is changed to `true`.
- **VERIFIED:** The file is checked again and the relevant test completes successfully.

Saving the file successfully only shows that the change was written. A relevant test or check is still needed to show that the feature works as expected.

## Verify the artifact and environment together

The same file can behave differently in different environments. Software versions, operating systems, dependencies, or configuration settings may affect the result.

For important checks, record:

- the file or version that was tested;
- the environment where the test ran;
- the check or test that was performed;
- the source of the verification result.

## RESULT_VERIFIED and PATH_VERIFIED

Verification can answer two separate questions:

1. **RESULT_VERIFIED** — Did the expected result occur?
2. **PATH_VERIFIED** — Were the required steps and checks completed on the way to that result?

For example, a generated file may look correct at first glance. If a required test was skipped, however, the required verification process is not complete.

## What can go wrong?

If observation, interpretation, and verification are not separated, an assumption may gradually be treated as verified information. A missing test or unchecked detail can then be overlooked.

## Core rule

> **A “done” status shows that the action ended. Verify the result separately before treating it as correct.**

## How to apply it

Where this verification rule should be placed depends on how the agent is being used. The same prompt does not need to be repeated in every message.

### 1. In the chat for a single task

If the agent is being used for one specific task, add the instruction directly to the chat message that assigns the task. This is useful for file changes, tests, builds, or other work that requires verification.

For example:

> **For this task, report OBSERVED, INTERPRETED, ACTION, and VERIFIED separately. Do not use the action itself as proof of success. In VERIFIED, include the test, file check, command result, or other evidence used to confirm the outcome. If verification was not performed, state that clearly.**

A separate file is not required in this case. The instruction can apply only to that task in the chat.

### 2. As a persistent rule for a project

If the same verification rule should apply to many tasks in a project, place it in a persistent project instruction that the agent is actually configured to read.

The filename depends on the tool. Some tools may read an `AGENTS.md` or another dedicated project instruction file. The rule can also be documented in a normal `README.md`, but **do not assume that every agent automatically reads the README as task instructions**. First confirm which instruction files the tool actually loads.

With this approach, the user assigns the task without repeating the same verification prompt every time. The agent reads the project instructions and applies the rule.

### 3. In an automated agent workflow

If a workflow or orchestrator starts the agent automatically, make the verification rule a persistent part of the task instructions sent to the agent.

The system can require OBSERVED, INTERPRETED, ACTION, and VERIFIED to be produced as separate fields. Depending on the system, these values can be stored in JSON output, a task record, or a shared workspace file that the agents can access. A simple “done” message is then not treated as proof of success.

For example, an automated system could produce an output like this:

```json
{
  "OBSERVED": "The configuration file contains feature_enabled: false.",
  "INTERPRETED": "The feature is disabled according to the current setting.",
  "ACTION": "The feature_enabled value was changed to true.",
  "VERIFIED": "The file was checked again and the relevant test completed successfully."
}
```

Because each piece of information is stored separately, the system can distinguish the action from the verification result. If the `VERIFIED` field is empty or states that verification was not performed, the task should not be treated as verified only because the agent reported “done.”

### When should the user apply this rule?

Use it especially when asking an agent to **change something or demonstrate that something works correctly**. Examples include file or code changes, tests, builds, configuration changes, tool use, and work that will later be handed to another agent.

For important tasks, also record the file or version that was checked and the environment in which verification was performed.

## When to use it

This distinction is especially useful when an agent changes files, runs tests, creates outputs, uses another tool, or hands work to another agent.

For low-risk and easily reversible changes, a shorter check may be enough. For example, after correcting a typo in a README file, checking that the updated text appears correctly may be sufficient; running a comprehensive test suite may not be necessary.
