# repo-audit

repo-audit is a lightweight skill for inspecting a repository's git history before reading implementation files.

It packages a simple five-command workflow inspired and turns it into a reusable skill for AI agents and humans doing first-pass codebase triage.

## What it does

repo-audit helps answer a few high-value questions quickly:

- Which files change the most?
- Who has historically carried the codebase?
- Where do bug-fix commits cluster?
- Is delivery steady, slowing down, or bursty?
- How often does the team appear to be firefighting?

The goal is not to replace reading code. The goal is to decide where to read first and what risks to expect.

## Install

Place the `SKILL.md` file into the following location:

```bash
~/.copilot/skills/repo-audit/
```

In a new copilot session, run `/skills` and `r` to refresh the installed skills list.

## Skill path

The publishable skill lives at:

- `skills/repo-audit/SKILL.md`

## When to use it

Use repo-audit when:

- entering an unfamiliar repository
- doing lightweight codebase triage
- deciding which files or subsystems to inspect first
- estimating maintenance risk before making changes
- performing an initial engineering audit

## Example prompts

- "Run a audit on this repo before we read the code."
- "Use repo-audit and tell me which files I should inspect first."
- "Audit this repository's history for hotspots, ownership concentration, and firefighting signals."

## What the skill contains

The skill currently stays close to the original article and includes:

- the five core git commands
- guidance for interpreting each command's output
- caveats around squash merges, weak commit messages, monorepos, and noisy history
- a synthesis pattern for turning raw command output into a short triage summary

## Design principles

- Keep the workflow transparent and easy to audit.
- Prefer plain git commands over premature automation.
- Treat repository history as signal, not proof.
- Preserve clear attribution to the original source idea.
- Optimize for general-purpose use across many repositories.

## Licence

MIT
