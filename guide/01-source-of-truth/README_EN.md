# Source of Truth

[🇹🇷 Türkçe](README.md)

## The problem
An agent can produce a perfectly reasonable change against the wrong version of a file.

Imagine Agent A reads version 4. Agent B then creates version 5. Agent A does not notice and writes its change using the old state. Nothing about Agent A's reasoning has to be bad for the result to be wrong.

## Plain-language rule
> **Before changing something, know which version is the source of truth—and confirm it is still the same version immediately before writing.**

## Practical pattern
Keep a small record for critical work: SOURCE, VERSION, STATUS, VERIFIED_BY, and a HASH or another stable version identifier.

Before a write:
1. Read the current artifact.
2. Record its version/hash.
3. Prepare the change.
4. Re-check the live version/hash.
5. If it changed, stop and re-read instead of overwriting.

## Synthetic example
Two agents work on a fictional file called `settings.yaml`.

Agent A reads hash `abc123`. Agent B updates the file, producing hash `def456`. Before Agent A writes, it checks again and sees `def456`.

**Correct behavior:** stop, reload, and rebuild the change.

**Unsafe behavior:** write the patch prepared for `abc123` over `def456`.

## When to use it
Use this whenever multiple people, agents, branches, automations, or machines can change the same artifact.

For a tiny single-user experiment with disposable files, the full record may be unnecessary.