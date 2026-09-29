# Agent Handoffs

[🇹🇷 Türkçe](README.md)

## The problem
Agent A can finish its task correctly and Agent B can still fail because an important constraint disappeared during the handoff.

The failure is not always inside an agent. Sometimes it is **between** agents.

## Plain-language rule
> **A handoff should say what changed, what must not change, what was actually verified, and what was not checked.**

## Minimal handoff receipt
TASK_ID · INPUT_VERSION · OUTPUT_VERSION · CHANGED · PRESERVE · VERIFIED · SKIPPED_CHECKS · NEXT_AGENT

## Coverage matters
“Checked everything” is not evidence of coverage. For bounded tasks, record EXPECTED_ITEMS, OBSERVED_ITEMS, CHECKED_ITEMS, SKIPPED_ITEMS, and COVERAGE_STATUS.

If 12 files were expected and only 10 were inspected, the receipt should make that visible.

## Synthetic example
Agent A updates three synthetic configuration files but tests only two. A useful handoff says: 3 files changed; 2 tested; 1 not executed; reason: required runtime unavailable.

Agent B now knows exactly where verification must continue.

## When to use it
Use structured handoffs when work crosses agents, sessions, machines, or people. For a one-step disposable task, a formal receipt may be overkill.