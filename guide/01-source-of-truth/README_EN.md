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
5. Confirm that the files affected by the change and required for recovery have a restorable version.
6. If the file changed in the meantime, stop, review the current version, and prepare the change again.

## Rollback Point

A backup should not mean creating unnecessary copies of every file. Protect the files that will be directly affected by the change and are needed to return to the previous state if something goes wrong.

**For projects using Git:** Check the working tree before making the change. Confirm that the version you may need to restore exists in commit history. If important changes have not yet been recorded, protect them with a safe commit, branch, or another appropriate Git method before overwriting anything.

**For projects without Git:** Copy the current version of the file that will be changed to a separate backup directory. Include a timestamp or version in the filename so it is clear which backup was created before which change. For example: `settings.before-change.2026-09-29.yaml`.

The existence of a backup alone is not enough. It should be clear which file can be restored, and the backup should be stored separately from the location affected by the change.

> **The goal is not to create as many backups as possible. The goal is to create the recovery point needed before a risky change.**

## Example

Two agents are working on a fictional `settings.yaml` file.

Agent A opens the version with hash `abc123`. Agent B then updates the file, changing the hash to `def456`. Before saving its work, Agent A checks the file again and sees that it has changed.

**Correct action:** Review the current file and prepare the change against the new version.

**Risky action:** Apply the change prepared for `abc123` directly over the newer `def456` version.

## How to apply it

This method can be added to an AI agent's task instructions when the agent works with files or code, especially when another agent, person, or automation may change the same files.

A starting instruction can be:

> **Before making a change, check the current file version and hash. Confirm that the files affected by the change and required for recovery have a restorable version. If you have file-system or Git access, create the required rollback point yourself and verify that it was created. If you do not have that access, do not assume a backup exists; ask the user to create a rollback point or stop the change. Check the file again immediately before saving. If the version or hash has changed, do not overwrite it. Review the current version and prepare the change again.**

This instruction can be used with ChatGPT, Claude, Codex, or similar agents that work with files and code. If the agent has permission to work with the file system or Git, it can also create the backup or rollback point itself. If it does not have that permission, it should not treat the step as completed.

In more automated workflows, this check should not rely only on a prompt. The system can compare the file's version or hash before work begins and immediately before the change is saved. If the values differ, the write can be stopped and the agent can be required to reload the current file.

## When to use it

This check is important when the same file can be modified by multiple people, agents, branches, or automations, or when it can be updated from different working environments.

For small experiments where one person works alone and changes can easily be discarded, a lighter check may be enough.
