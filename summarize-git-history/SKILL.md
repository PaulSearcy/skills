---
name: summarize-git-history
description: Summarize a git repository's commit history with flexible filters. Use when the user asks to review git history, summarize commits, see what was done in a repo, or asks about work over a time period, last N commits, or by author.
---

# Summarize Git History

Analyze git log output and produce a thematic summary of work done in a repository, respecting user-specified filters.

## When to Use

- User asks "what did I do in this repo" or "summarize my commits"
- User asks about work in a date range ("in 2025", "last month", "since March")
- User asks for "last N commits"
- User wants a summary filtered by author, branch, or path

## Workflow

### 1. Determine Filters

Parse the user's request for these optional filters. Apply only what's mentioned; omit the rest.

| Filter | git log flag | Example user input |
|--------|-------------|-------------------|
| Author | `--author="name"` | "my commits", "what did Dan do" |
| Date range | `--after="YYYY-MM-DD" --before="YYYY-MM-DD"` | "in 2025", "last 3 months", "since March" |
| Count | `-n N` | "last 10 commits" |
| Branch | `branch-name` | "on the feature branch" |
| Path | `-- path/` | "in the src/ folder" |
| Exclude merges | `--no-merges` | Default: always exclude merges |

**Date shortcuts** (resolve relative to today's date):

- "in YYYY" → `--after="YYYY-01-01" --before="YYYY+1-01-01"`
- "last N months" → `--after` = N months ago
- "this year" → `--after` = Jan 1 of current year
- "last quarter" → compute start/end from current date

**Author inference**: If the user says "I" or "my", try to infer their identity from `git log` recent commits or the repo's git config. Use a loose `--author` grep pattern that covers name and email.

### 2. Run git log

Run two commands in parallel:

**a) Oneline list** (for the summary):

```bash
git log --oneline --no-merges [filters]
```

**b) Monthly stats** (for activity distribution):

```bash
git log --no-merges --format="%ai" [filters] | cut -d'-' -f1-2 | sort | uniq -c | sort -k2
```

Also grab total commit count from (a).

### 3. Produce the Summary

Structure the output as:

```
**N commits** across M months (date range or "last N" context).

**Theme 1 (e.g. "eCode Integration")**
- Bullet point per logical unit of work
- Group related commits, don't list every commit individually

**Theme 2 (e.g. "Infrastructure / CI")**
- ...

**Theme N**
- ...
```

**Grouping rules**:

- Read through all commit messages and cluster by **feature area or intent**, not chronology.
- Typical themes: new features, bug fixes, refactoring, tests, CI/CD, dependencies, documentation, cleanup/lint.
- Merge tiny themes (< 3 commits) into a "Misc" bucket unless they're notable.
- Within each theme, order bullets roughly by significance, not date.
- Use the commit messages to infer *what was done*, not just parrot them. Summarize in plain language.
- If a single feature spans many commits (e.g. 10 commits building out an integration), collapse into one or two bullets.

**Tone**: Direct, factual, no filler. Write as a peer summarizing work for a standup or a year-end review.

### 4. Optional: Detailed Breakdown

If the user asks for more detail on a specific theme, re-run git log with a `--grep` or path filter and provide a deeper dive with file-level changes:

```bash
git log --stat --no-merges --grep="keyword" [filters]
```
