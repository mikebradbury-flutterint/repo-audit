---
name: repo-audit
description: Check a repo’s git history before opening source files. Use it when exploring an unfamiliar codebase, triaging a repo, or choosing where to look first. Five git commands reveal churn hotspots, contributor concentration, bug-prone files, delivery pace, and firefighting patterns.
licence: MIT
metadata:
    author: mikebradbury-flutterint
    version: "0.1.0"
    tags:
        - git
        - codebase-analysis
        - triage
        - diagnostics
        - software-engineering
---

# repo-audit

Use git history to build a diagnostic picture of a repository before opening implementation files. The purpose is not to replace code reading. The purpose is to decide where to look first and what risks to expect.

## When to use this skill

Use this skill when:

- you just entered an unfamiliar repository
- you want a fast first-pass repo triage
- you need to choose which files to inspect first
- you want to estimate maintenance risk before making changes
- you are auditing a codebase at a high level

Do not use this skill as proof of code quality by itself. These commands provide signals, not verdicts.

## Prerequisites

Before running the commands:

1. Confirm you are inside a git repository.
2. Prefer the repository root so path output is easier to read.
3. If the repo is extremely large or a monorepo, interpret results with extra caution.
4. If useful, adjust the time window from `1 year ago` to match the repository's pace.

Basic repo check:

```bash
git rev-parse --show-toplevel
```

## Workflow

Run the five commands below in order. After each one, interpret the output before moving on.

### 1) What changes the most

```bash
git log --format=format: --name-only --since="1 year ago" | sort | uniq -c | sort -nr | head -20
```

What it shows:

- the most frequently touched files in the chosen time window

How to interpret it:

- high churn can mean active development, not necessarily poor quality
- high churn plus team fear is often a codebase drag signal
- the top few files are strong candidates for first inspection
- cross-reference the top 5 against the bug hotspot command below

What to say about it:

- call out likely hotspots, not just the top file
- note whether churn seems concentrated or spread out
- distinguish active product work from suspicious repeated patching when possible

### 2) Who built this

```bash
git shortlog -sn --no-merges HEAD
```

Optional recency check:

```bash
git shortlog -sn --no-merges --since="6 months ago" HEAD
```

What it shows:

- contributors ranked by commit count
- possible contributor concentration and maintenance handoff risk

How to interpret it:

- if one person dominates the history, bus factor may be low
- if historic top contributors are absent from recent history, handoff risk may be real
- compare lifetime contribution with recent contribution, not just one or the other

Important caveat:

- squash-merge workflows can make this reflect who merged, not who wrote the code

What to say about it:

- describe concentration carefully
- avoid overclaiming ownership from commit count alone
- explicitly mention squash-merge distortion if it may apply

## Monorepo de-noising

If raw churn is dominated by repository metadata, generated artifacts, or bulk-content paths, do a filtered second pass before deciding what code to read.

Treat these as likely noise unless the user is explicitly auditing build, release, localization, or docs workflows:

- lockfiles and dependency manifests
- changelogs and release metadata
- snapshots, generated files, coverage, dist, and build outputs
- bulk-content paths such as docs or localization trees

### Required heuristic

If 5 or more of the top 10 churn results are likely noise, rerun both churn and bug-hotspot analysis with excludes.

If the filtered pass is still dominated by bulk-content paths such as `docs/`, `locales/`, or `i18n/`, rerun once more with an additional repo-specific exclude and label it clearly as a runtime-focused pass.

### Directory-level triage

Use this when a repo is large enough that file-level hotspots are too noisy at first glance:

```bash
git log --format=format: --name-only --since="1 year ago" | grep . | cut -d/ -f1-2 | sort | uniq -c | sort -nr | head -20
```

### Filtered churn pass

```bash
git log --format=format: --name-only --since="1 year ago" | \
grep -vE '^$|(^|/)(package\.json|pnpm-lock\.yaml|package-lock\.json|yarn\.lock|bun\.lockb?|CHANGELOG\.md|.*\.snap)$|(^|/)(__snapshots__|dist|build|coverage|generated|vendor)/' | \
sort | uniq -c | sort -nr | head -20
```

### Filtered bug-hotspot pass

```bash
git log -i -E --grep="fix|bug|broken" --name-only --format='' | \
grep -vE '^$|(^|/)(package\.json|pnpm-lock\.yaml|package-lock\.json|yarn\.lock|bun\.lockb?|CHANGELOG\.md|.*\.snap)$|(^|/)(__snapshots__|dist|build|coverage|generated|vendor)/' | \
sort | uniq -c | sort -nr | head -20
```

### Reporting rule

When you use a filtered pass:

- report the raw pass and the filtered pass
- explicitly name the excluded categories or paths
- say why the filter was necessary
- distinguish generic de-noising from any repo-specific runtime-focused pass

### 3) Where do bugs cluster

```bash
git log -i -E --grep="fix|bug|broken" --name-only --format='' | sort | uniq -c | sort -nr | head -20
```

What it shows:

- files most often associated with bug-related commit messages

How to interpret it:

- files appearing on both this list and the churn list are especially worth inspecting
- repeated bug-related touches can indicate fragile areas, unclear design, or missing tests
- weak commit-message discipline reduces the signal sharply

What to say about it:

- highlight overlap with churn hotspots
- call this a heuristic based on commit language, not a definitive defect map

### 4) Is this project accelerating or dying

```bash
git log --format='%ad' --date=format:'%Y-%m' | sort | uniq -c
```

What it shows:

- commit count by month across the repository history

How to interpret it:

- a steady rhythm often suggests consistent delivery
- major drops may correlate with staffing changes or priority shifts
- spikes followed by quiet stretches may suggest batch releases or crisis bursts
- this is team/process signal as much as code signal

What to say about it:

- describe the shape of the curve, not just the raw counts
- avoid simplistic judgments like "dead repo" without context

### 5) How often is the team firefighting

```bash
git log --oneline --since="1 year ago" | grep -iE 'revert|hotfix|emergency|rollback'
```

What it shows:

- visible crisis-related commits in the chosen window
- an empty result is meaningful here and does not automatically indicate a command problem

How to interpret it:

- a few may be normal
- frequent reverts or hotfixes can point to deployment instability, weak tests, or release-process issues
- zero matches can mean stability, or just vague commit messages

What to say about it:

- talk about operational confidence, not only code defects
- treat this as a pattern detector, not a precise metric

## Synthesis

After running all five commands, produce a short triage summary with this shape:

1. Top hotspots
    - which files appear risky or central
2. Ownership risk
    - whether knowledge looks concentrated or transitioned
3. Bug overlap
    - whether churn and bug language converge on the same files
4. Delivery pattern
    - steady, declining, bursty, or hard to tell
5. Firefighting signal
    - visible crisis patterns, ambiguous, or quiet
6. Where to read first
    - name the first few files or subsystems to inspect next

## Suggested response style

When using this skill, give the user:

- the exact commands you ran
- the key results, compressed into a readable summary
- caveats that materially affect interpretation
- a recommendation for what code to inspect next

Avoid:

- pretending these metrics are objective truth
- using churn alone as a synonym for bad code
- making personnel claims stronger than the evidence supports
- forcing certainty when the repo history is noisy or ambiguous
- reporting filtered hotspots without also saying what was excluded

## Common pitfalls

- Monorepo noise: lockfiles, manifests, snapshots, docs, and localization trees can dominate the first pass.
- Merge-strategy distortion: squash merges can hide true authorship.
- Commit-message distortion: vague messages weaken bug and firefighting signals.
- Time-window distortion: `1 year ago` may be too long or too short.
- File-path noise: renames and generated files can muddy hotspot counts.
