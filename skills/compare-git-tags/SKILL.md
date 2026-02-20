---
name: compare-git-tags
description: Identify and summarize changes between git tag versions for a dependency upgrade, starting from an npm package name + installed version. Use when the user asks what changed between versions/tags, wants upgrade notes for major bumps (e.g. 10.x -> 12.x), or wants a tag-to-tag diff and release summary.
---

# Compare Git Tags (npm-first)

Summarize what changed between two versions of a dependency by resolving the upstream git repo and comparing **tags**. Prefer **GitHub Releases/CHANGELOG** for narrative, backed by **git log/diff** for ground truth. Output both **upgrade notes** and a **thematic summary**.

## When to Use

- “We have `pkg@10.1.0`; what changed in `12.x`?”
- “Compare tags `v10.1.0` → `v12.4.0` and call out breaking changes”
- “Give migration/upgrade notes for this dependency”

## Inputs (infer from context)

- **Package name** (required): e.g. `@apidevtools/swagger-parser`
- **From version**: default to the installed version
- **To version**:
  - exact version/tag (e.g. `12.0.0`), or
  - “latest in major N” (e.g. “latest 12.x”)
- Optional: **path scope** (subdir), **include merges** (default: no), **only releases** (skip diffs)

## Workflow

### 1) Resolve installed version (npm-first)

From the consuming repo (your app), determine the installed version:

```bash
npm ls <pkg> --json
```

If the user already told you the version (e.g. “we have 10.1.0”), use that as **from**.

### 2) Resolve upstream repository URL

Prefer resolving from the installed package metadata:

- Read `<pkg>/package.json` and inspect `repository` / `homepage` / `bugs` fields.
  - If `repository` is an object, use `.url`.
  - Normalize common prefixes: `git+https://…` → `https://…`, strip `.git`.

Fallbacks (use if metadata is missing/ambiguous):

- `npm view <pkg> repository --json` (network)
- If a GitHub URL is already provided by the user, use it directly.

### 3) Clone/fetch and resolve tags

Clone the upstream repo (or reuse an existing checkout), then:

```bash
git fetch --tags --force
```

Resolve **from tag**:

- Try `v<from>` then `<from>` (many repos prefix tags with `v`).

Resolve **to tag**:

- If user specified exact version: try `v<to>` then `<to>`.
- If user specified “latest in major N”:

```bash
git tag --list "vN.*" --sort=-v:refname | head -n 1
git tag --list "N.*"  --sort=-v:refname | head -n 1
```

If the repo uses unconventional tags, list candidates and pick the best semantic match:

```bash
git tag --sort=-v:refname | head -n 50
```

### 4) Gather narrative sources (preferred)

Look for release notes and changelogs covering the range:

- GitHub releases (preferred when available):

```bash
gh release list -R owner/repo --limit 50
gh release view <tag> -R owner/repo
```

- Repo changelog files (if present): `CHANGELOG.md`, `HISTORY.md`, `RELEASE_NOTES.md`.

### 5) Ground truth: diff + log between tags

Run these between `FROM_TAG..TO_TAG`:

```bash
git log --no-merges --oneline FROM_TAG..TO_TAG
git diff --stat FROM_TAG..TO_TAG
git diff --name-only FROM_TAG..TO_TAG
```

If the user wants to focus on a sub-area:

```bash
git log --no-merges --oneline FROM_TAG..TO_TAG -- path/
git diff FROM_TAG..TO_TAG -- path/
```

### 6) Identify upgrade-impact signals

Prioritize items that affect consumers:

- **Breaking changes**: `BREAKING`, `!:`, `remove`, `rename`, `drop`, `deprecated`, `migration`
- **Runtime changes**: Node engine/support matrix, ESM/CJS, TypeScript types, peer deps
- **Behavior changes**: parsing/validation semantics, defaults, error types/messages
- **Security**: CVEs, dependency bumps that require action

### 7) Output (two sections)

#### A) Upgrade notes (actionable)

- **Target**: FROM_TAG → TO_TAG (and mention majors crossed, e.g. 10→11→12)
- **Breaking changes**
  - What changed
  - What you need to do
  - Evidence: release note section or commit/PR reference
- **Behavior changes**
- **Deprecations / removals**
- **Dependency / runtime changes**
- **Recommended upgrade path**
  - If crossing majors, prefer stepping through major boundaries (10→11→12) and highlight each major’s headline changes.

#### B) Release summary (thematic)

Cluster changes into 4–8 themes (typical buckets):

- Parsing/validation behavior
- API surface / exports
- Types/TypeScript
- CLI/tooling
- Performance/refactors
- Docs/examples
- Tests/CI
- Dependencies/security

Rules:

- Don’t list every commit; group into meaningful bullets.
- Prefer release notes/changelog wording; use `git` output to validate and add specifics.

## Example prompt this skill should handle

“Compare `@apidevtools/swagger-parser` from `10.1.0` to the latest `12.x` and give me upgrade notes + a thematic summary.”

