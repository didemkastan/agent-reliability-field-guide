# Agent Reliability Field Guide

## 📚 Start Reading

Click a topic below to open its content.

- [1. Source of Truth](guide/01-source-of-truth/README_EN.md) — Make sure work is based on the correct file and current version.
- [2. Agent Handoffs](guide/03-agent-handoffs/README_EN.md) — Transfer work to another agent together with the information it needs.
- [3. Verification](guide/04-verification/README_EN.md) — Understand the difference between reporting “done” and actually verifying the result.
- [4. Handoff Receipt Template](patterns/handoff-receipt.md) — A ready-to-use structure for recording a handoff.

> This list will be updated as new sections are added.

[🇹🇷 Türkçe](README.md)

> **Reliable AI agents are not agents that never make mistakes. They are systems that can show what happened, why it happened, what was verified, and what remains uncertain.**

This guide was created to support more reliable, traceable, and verifiable AI-agent workflows. It is available in Turkish and English so the content can reach and be used by a wider community.

## Purpose of this guide

Agent systems can run into problems for many reasons. An agent may work on an outdated file, two agents may affect each other's changes, a retry may repeat the same incorrect assumption, or a “done” status may be accepted without enough verification.

This guide presents practical methods that can help prevent and manage these situations.

Each method follows the same structure:

**Problem → Explanation → Example → What can go wrong? → Practical method → Verification → When it may be unnecessary**

## Core areas

1. **Source of truth** — provenance, canonical versions, and decision records.
2. **Safe changes** — preventing changes based on outdated information or the wrong version.
3. **Agent handoffs** — transferring required information and making completed and skipped checks visible.
4. **Verification** — separating observations, interpretations, actions, and verification results.
5. **Retries** — retrying with information from the previous failure and stopping when needed.
6. **Memory & context** — managing temporary information, changed sources, incorrect records, and conflicts.
7. **Security & authority** — limiting permissions to what is needed and rechecking authority when necessary.
8. **Observability** — recording actions and verification results so they can be reviewed later.

## Safety & privacy

This repository does **not** publish private project data, company information, customer data, internal code, private conversations, credentials, or confidential logs.

Examples are synthetic or generalized. Public ideas may inspire a method, but the value of this guide comes from bringing ideas together, evaluating them, testing them, and making them practical without exposing private or proprietary material.

## Status

🚧 **The first version of the guide is in progress.** New sections will be added based on need and practical use.

## Guiding principle

> **Do not use a method only because it is new. When a need appears, test it on a small scale, verify its benefit, and add it to the workflow when appropriate.**

## Language

- English: this file
- Türkçe: [README.md](README.md)

## License

License selection will be finalized before the first stable release.
