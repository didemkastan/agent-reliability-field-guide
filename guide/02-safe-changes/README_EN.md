# Safe Changes

[🇹🇷 Türkçe](README.md)

> This section teaches one thing: **when you ask an agent to make a change, how do you make sure only the requested thing changes?**
> Agent handoffs and automation are covered separately.

## How to read the boxes on this page

- 👁 **Example:** Read only.
- 📋 **Paste to the agent:** Put it in the agent's chat or prompt.
- 📄 **Write to file:** Put it in the named file.
- 💻 **Run in Terminal:** Run it in Terminal (macOS/Linux) or PowerShell (Windows).

## A few terms

- **Repository (repo):** The workspace that stores project files and change history.
- **Commit:** A recorded snapshot of changes.
- **Diff:** A line-by-line comparison showing what was removed and added.
- **Scope:** The boundary of the task: what the agent may change and what it must preserve.

## Problem

When you ask an agent for a small change, it may modify other areas while completing the request.

For example, you ask it to fix one link, but it also rewrites nearby text or touches unrelated files. The link is fixed, but the change is now broader than requested.

## Core rule

> **Before making a change, define what may change and what must be preserved. Make the smallest change required for the requested result.**

## Change boundary: three fields

- **CHANGE:** What you want changed.
- **PRESERVE:** What must remain unchanged.
- **DO NOT:** What must not be done in this task.

📋 **Paste to the agent**

```
CHANGE:
- Replace the old link in README.md with the new one.

PRESERVE:
- Other text
- Heading order
- File structure

DO NOT:
- Change other files.
- Rewrite unrelated text.
```

The list does not need to be long. For a small task, write only the critical boundaries.

## Smallest necessary change

If a typo can be fixed on one line, there is no need to rewrite the paragraph or reformat the file at the same time.

If the agent notices another issue outside the scope, it should **report it** rather than fix it automatically. The user decides whether to expand the scope.

## Example: expected vs. actual

Request: change one link in the README.

👁 **Example**

```
EXPECTED                ACTUAL
README.md               README.md
                        config.yaml     ← unexpected
                        package.json    ← unexpected
```

Even if the link was fixed correctly, the other two files are outside the task. Unexpected changes should be reviewed and reverted if they are not needed.

## How do you check the scope?

After the task, ask two questions:

1. **Which files changed?** Does the list match what you expected?
2. **What changed inside those files?** Was a protected area touched?

File count alone is not enough. The wrong line can be changed inside the right file.

### Check on GitHub

On the Pull Request page, open **Files changed**. It shows the files and lines changed by the PR.

### On your computer: changes not committed yet

From the repository folder:

💻 **Run in Terminal**

```bash
git status
git diff --stat
git diff
```

- `git status` shows the state of the working tree.
- `git diff --stat` summarizes uncommitted changes by file.
- `git diff` shows the line-level diff for uncommitted changes.

> ⚠️ If the agent already committed the change, a normal `git diff` may be empty. That does not mean no change was made.

### If the change was already committed

To inspect the latest commit:

💻 **Run in Terminal**

```bash
git show --stat HEAD
git show HEAD
```

- `git show --stat HEAD` summarizes files changed in the latest commit.
- `git show HEAD` shows its line-level diff.

If you are using a PR, GitHub's **Files changed** tab is usually the easiest check.

### Before reverting an unexpected change

If the change is **not committed** and you are certain you want to discard all local changes in that file:

💻 **Run in Terminal**

```bash
git restore config.yaml
```

> ⚠️ `git restore config.yaml` can permanently discard uncommitted changes in that file. If you are unsure, do not run it; inspect `git diff config.yaml` first.
>
> If the change is already committed, do not use this command as the solution. Inspect the commit diff first and choose an appropriate revert method separately.

## Ready-to-use prompt

📋 **Paste to the agent**

```
Before starting, define the scope: state which files or sections may
change and what must be preserved.

Make only the smallest change required to complete the task.
Do not change unrelated files, text, settings, dependencies, or
existing working behavior.

If you notice a problem outside the scope, report it instead of fixing it.
If completing the task requires expanding the original scope, stop before
making that change and explain why.

When finished, compare the actual changes with the original scope.
Check the diff when possible and explicitly report any out-of-scope change.

Return:
CHANGED: what actually changed
PRESERVED: what you checked remained unchanged
UNEXPECTED_CHANGES: unexpected changes (or: none)
SCOPE_STATUS: did the change stay within the task boundary?
```

> **The agent's report is not proof by itself.** Verify it against the real diff whenever possible.

## Where do shared rules go when using multiple agents?

Instead of repeating permanent rules in every task, keep shared rules in one project instruction source. In a simple setup, you can use `AGENTS.md` at the repository root.

👁 **Example**

```
PROJECT/
├── AGENTS.md
├── src/
└── ...
```

📄 **Write to file:** `AGENTS.md`

Put the safe-change rules and permanent project boundaries in this file as normal Markdown text.

> ⚠️ **A file existing in the repository does not mean every agent automatically reads it.** Tools load project instructions differently, and those mechanisms can change over time.

| Tool | Default project instruction | Connecting shared rules |
|---|---|---|
| Codex CLI | `AGENTS.md` | It can use the rules from the repository root. |
| Claude Code | `CLAUDE.md` | Connect the shared rules through Claude's project instructions or keep the relevant rules there. |
| Gemini CLI | `GEMINI.md` | Use `GEMINI.md` or configure the shared filename with `context.fileName`. |
| Chat interfaces | Varies by tool | Add project instructions or provide the required file to the conversation. |

After setup, use each tool's current mechanism to check which instructions it actually loaded.

### How do you check whether the instruction file was loaded?

You can place a short, unique marker in `AGENTS.md`:

📄 **Write to file:** `AGENTS.md`

```
RULE-VERSION: v1.3
```

At the beginning of a new task, ask the agent to reproduce that line exactly.

> If it cannot reproduce the line correctly, do **not assume** the instruction file was loaded. Reproducing it is a useful initial check, but it is not conclusive proof for critical work. When needed, also record the instruction file version or hash.

Once shared rules are connected correctly, each task only needs its task-specific boundaries:

👁 **Example**

```
SHARED / PERSISTENT → project instruction
  safe-change rules
  permanent project boundaries

TASK-SPECIFIC       → task message
  TASK / CHANGE / PRESERVE / DO NOT
```

## When to use it

- When making a limited fix to working code or files
- When specific areas must remain unchanged
- When an agent should touch only specific files

Detailed scope records may not be necessary for small experiments that are easy to discard.

## Summary

1. Write **CHANGE / PRESERVE / DO NOT** before work starts.
2. Ask for the **smallest change** and a short scope report.
3. Distinguish uncommitted from committed changes when inspecting them.
4. Verify the agent's report against the **diff** whenever possible.
5. Do not automatically treat unexpected changes as part of the task.

## Next steps

- **Agent Handoffs:** Task packages, version checks, and work with external agents.
- **Agent Automation:** Automated triggers and agent transitions are covered separately.
