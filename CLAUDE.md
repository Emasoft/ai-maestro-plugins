# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A **Claude Code plugin marketplace index** — not plugin source code. The actual plugins live in their own GitHub repos (all under `Emasoft/ai-maestro-*`). The only thing this repo ships is `.claude-plugin/marketplace.json`, which Claude Code reads after `claude plugin marketplace add https://github.com/Emasoft/ai-maestro-plugins`.

There is no build step, no test suite, no application code. Edits here are either:
1. **Automated** — version bumps from upstream plugin releases (see "Version-bump pipeline" below).
2. **Manual** — adding/removing a plugin entry, fixing description/category/scope metadata.

## The single source of truth

`.claude-plugin/marketplace.json` defines every available plugin. Each `plugins[]` entry has:

- `name` — install identifier (`claude plugin install <name> ai-maestro-plugins`)
- `source.url` — git URL of the upstream plugin repo
- `version` — must match the upstream's `.claude-plugin/plugin.json` `version` field
- `category` — `core-plugin` (installs to user/project scope) or `role-plugin` (installs to local scope, one per agent)
- `scope` — `local` for role-plugins; omitted/`project` for core-plugins
- `repository` — used by the workflow to fetch plugin.json via `gh api`

When adding a new plugin, copy an existing entry as the template and keep the field order consistent — the workflow's Python parser is forgiving but `git diff` reviews are easier with stable ordering.

## Version-bump pipeline

`.github/workflows/update-plugin-version.yml` is the only CI. It fires on:

1. **`repository_dispatch` type `plugin-updated`** — sent automatically by each plugin repo's own `notify-marketplace.yml` workflow when it tags a release. Payload carries `plugin` (name) and optionally `version`.
2. **`workflow_dispatch`** — manual trigger via `gh workflow run update-plugin-version.yml -f plugin=<name>`. Use this when an upstream notify failed or you want to force-resync.

The flow:
- If the dispatch payload omits `version`, the workflow calls `gh api repos/<owner>/<repo>/contents/.claude-plugin/plugin.json` and pulls the version field directly.
- A Python heredoc updates the matching `plugins[]` entry in `marketplace.json` in-place (preserves indent=2, trailing newline).
- If `git diff` shows no change, exits cleanly. Otherwise commits as `chore: bump <plugin> to <version> [skip ci]` and pushes (with up-to-3 retry + `pull --rebase` on conflict).

**Concurrency guard**: `concurrency.group: update-plugin-version`, `cancel-in-progress: false` — two plugins released simultaneously serialize, never collide on `marketplace.json`.

**Required secret**: `MARKETPLACE_PAT` — GitHub PAT with `contents: write` on this repo. Used for the checkout fetch (`persist-credentials: false`, so it is never stored in `.git/config`), for the explicit `git push` in the final step (the PAT belongs to the repo owner, whose admin role bypasses the `baseline-pr-and-checks` ruleset on `main`), and for the cross-repo `gh api` call when fetching plugin.json. If you see `MARKETPLACE_PAT errors in CI`, the token is missing or expired in repo settings → Secrets.

## Working with marketplace.json

- **Validate before commit**: `python3 -c "import json; json.load(open('.claude-plugin/marketplace.json'))"` — a syntax error here breaks every user's `claude plugin marketplace update`.
- **Don't edit version manually unless the workflow is broken** — manual bumps drift from upstream and confuse the dispatch flow. If you must, mirror the upstream's `plugin.json` exactly.
- **The `metadata.version` at the top of marketplace.json refers to the marketplace schema/contract, not any individual plugin** — bump it only when the marketplace structure itself changes.

## Conventions enforced by commit history

- Bump commits use `chore: bump <plugin> to <version> [skip ci]` exactly. The `[skip ci]` is load-bearing — without it, every bump would re-trigger downstream workflows.
- Manual docs/structure changes use conventional commit prefixes (`docs(...)`, `feat(...)`, `chore(...)`).
- The merge-base for all PRs is `main`; there are no long-lived release branches.
