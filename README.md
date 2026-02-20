# psearcy-cursor-skills

Rules and skills for my personal engineering workflows in Cursor.

## What’s included

- **Rules**: Code style and workflow guardrails under `rules/`
- **Skills**: Reusable, task-focused playbooks under `skills/`

Included skills:

- `skills/compare-git-tags`
- `skills/sql-generic`
- `skills/summarize-git-history`
- `skills/js-esm-functional-expressions`

## Local install

Install this repository as a local Cursor plugin (from the Plugins UI / “install from folder”), selecting the repo root (the folder containing `.cursor-plugin/plugin.json`).

## Development

- Add new rules as `.mdc` files in `rules/` (with YAML frontmatter).
- Add new skills in `skills/<skill-name>/SKILL.md` (with YAML frontmatter).

