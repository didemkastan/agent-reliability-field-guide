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

Ask the agent to report its work using separate observation, interpretation, action, and verification fields.

For example:

> **For this task, report OBSERVED, INTERPRETED, ACTION, and VERIFIED separately. Do not use the action itself as proof of success. In VERIFIED, include the test, file check, command result, or other evidence used to confirm the outcome. If verification was not performed, state that clearly.**

For important tasks, also record the version or artifact that was checked and the environment where the verification ran.

In automated workflows, these fields can be stored separately so that a status message cannot replace an actual test or verification result.

## When to use it

This distinction is especially useful when an agent changes files, runs tests, creates outputs, uses another tool, or hands work to another agent.

For low-risk experiments that can easily be reversed, a shorter check may be enough.
