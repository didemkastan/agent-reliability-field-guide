# Agent Reliability Field Guide

## 📚 Start Reading

Open the documents directly without browsing through folders:

- [1. Source of Truth](guide/01-source-of-truth/README_EN.md) — Make sure work starts from the correct file and version.
- [2. Agent Handoffs](guide/03-agent-handoffs/README_EN.md) — Transfer work between agents without losing important information.
- [3. Verification](guide/04-verification/README_EN.md) — Separate a “done” status from actual verification.
- [Handoff Receipt Template](patterns/handoff-receipt.md) — A ready-to-use structure for recording a handoff.

> This list will be updated as new sections are added.

[🇹🇷 Türkçe](README.md)

> **Reliable AI agents are not agents that never make mistakes. They are systems that can show what happened, why it happened, what was verified, and what remains uncertain.**

A practical, bilingual field guide for building AI-agent workflows that are easier to verify, trace, recover, and trust.

## Why this guide exists

Agent systems can fail in surprisingly ordinary ways: an agent edits an outdated file, two agents overwrite each other's work, a retry repeats the same broken assumption, or a confident “done” message is mistaken for verification.

This guide turns those failure patterns into simple engineering practices.

Each pattern follows a consistent structure:

**Problem → Explanation → Synthetic example → What can go wrong → Practical pattern → Verification → When it may be unnecessary**

## Core areas

1. **Source of truth** — provenance, canonical versions, decision records.
2. **Safe changes** — stale-state protection and approval-to-execution checks.
3. **Agent handoffs** — explicit context, coverage receipts, skipped checks.
4. **Verification** — separating observations, interpretations, actions, and evidence.
5. **Errors & retries** — informed retries, retry budgets, root-error targeting.
6. **Memory & context** — provisional memory, invalidation, tombstones, conflict handling.
7. **Security & authority** — least privilege, scoped authority, revalidation.
8. **Observability** — durable evidence and trustworthy execution traces.

## Safety & privacy

This repository does **not** publish private project data, company information, customer data, internal code, private conversations, credentials, or confidential logs.

Examples are synthetic or generalized. Public ideas may inspire a pattern, but the value of this guide is in synthesis, explanation, testing, and practical application—not copying private or proprietary material.

## Status

🚧 **Early field-guide build.** The structure is being developed incrementally. Patterns will be added only when they provide a distinct, practical benefit.

## Guiding principle

> **Do not adopt a pattern because it is new. Adopt it when a real need appears, test it on a small scale, verify the benefit, and only then make it part of the workflow.**

## Language

- English: this file
- Türkçe: [README.md](README.md)

## License

License selection will be finalized before the first stable release.
