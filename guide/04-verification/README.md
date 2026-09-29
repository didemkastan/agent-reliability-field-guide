# Verification

[🇹🇷 Türkçe](README_TR.md)

## Why verification matters

An agent saying “done” tells us what the agent believes happened. It does not prove that the expected result actually occurred.

A useful workflow keeps four things separate:

- **OBSERVED** — What was directly seen in a tool, file, test, or system result.
- **INTERPRETED** — What the agent thinks that observation means.
- **ACTION** — What was changed or executed.
- **VERIFIED** — What was checked after the action to confirm the result.

## Synthetic example

An agent changes a configuration file to enable a fictional feature.

- **OBSERVED:** The file currently contains `feature_enabled: false`.
- **INTERPRETED:** The feature appears to be disabled by this setting.
- **ACTION:** The value is changed to `true`.
- **VERIFIED:** The file is read again and the relevant test is run successfully.

The action and the verification are different steps. A successful file write does not automatically prove that the feature works.

## Verify the artifact and its environment

A result can depend on more than the file itself. Runtime version, operating system, dependencies, or configuration may change the outcome.

For important checks, record:

- the artifact or version that was tested;
- the environment in which it was tested;
- the check that was performed;
- the verifier or verification source.

## Result verification and path verification

Two questions are useful:

1. **RESULT_VERIFIED** — Did we get the expected result?
2. **PATH_VERIFIED** — Did we reach that result through the intended and acceptable process?

Example: a generated file may look correct, but if the required test was skipped, the result may be visible while the intended verification path is incomplete.

## What can go wrong?

If these steps are mixed together, an interpretation can slowly become treated as evidence. A confident status message may then hide a missing test or an unchecked assumption.

## Practical rule

> **Treat “done” as a status statement. Treat verification as a separate check supported by observable evidence.**

## When is this useful?

This distinction is especially useful when an agent changes files, runs tests, creates builds, calls external tools, or hands work to another agent.

For low-risk, disposable experiments, a lighter check may be enough.