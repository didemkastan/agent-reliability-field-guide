# Source of Truth

[🇹🇷 Türkçe](README.md)

## Problem

An agent may prepare a correct change and still produce the wrong result if it is working on an outdated version of a file.

Suppose Agent A is working on version 4. While that work is in progress, Agent B updates the file and creates version 5. If Agent A does not notice the update, it may finish its work using version 4. The prepared change may be correct on its own, but applying it to the newer file could overwrite new changes or cause data loss.

## Core rule

> **Before changing a file, identify the version being used as the source of truth. Immediately before saving the change, confirm that the file is still on that version.**

## Practical method

For important files, record information such as SOURCE, VERSION, STATUS, VERIFIED_BY, and HASH.

When making a change:

1. Check the current version of the file.
2. Record its version or hash.
3. Prepare the change.
4. Check the version or hash again immediately before saving.
5. If the file changed in the meantime, stop, review the current version, and prepare the change again.

## Example

Two agents are working on a fictional `settings.yaml` file.

Agent A opens the version with hash `abc123`. Agent B then updates the file, changing the hash to `def456`. Before saving its work, Agent A checks the file again and sees that it has changed.

**Correct action:** Review the current file and prepare the change against the new version.

**Risky action:** Apply the change prepared for `abc123` directly over the newer `def456` version.

## How to apply it

This method can be added to an AI agent's task instructions when the agent works with files or code, especially when another agent, person, or automation may change the same files.

A starting instruction can be:

> **Before making a change, check the current file version and record its version or hash. Check it again immediately before saving. If the version or hash has changed, do not overwrite the file. Review the current version and prepare the change again.**

This instruction can be used with ChatGPT, Claude, Codex, or similar agents that work with files and code.

In more automated workflows, this check should not rely only on a prompt. The system can compare the file's version or hash before work begins and immediately before the change is saved. If the values differ, the write can be stopped and the agent can be required to reload the current file.

## When to use it

This check is useful when multiple people, agents, branches, automations, or machines can modify the same file.

For small experiments where one person works alone and changes can easily be discarded, a lighter check may be enough.
