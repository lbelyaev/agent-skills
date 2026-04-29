---
name: ship
description: Use whenever the user asks to "ship", "open a PR", "land this", "send for review", "push and open a PR", or after they confirm a feature is done. Lands the current branch as a pull request — runs every available gate locally first, refuses to push if anything fails, then commits, pushes, and opens a PR with a body derived from the branch's commit history. Project-agnostic; discovers gates from package.json scripts.
allowed-tools: [Bash, Read, Grep, Glob]
---

# Ship the current branch

Land the active feature branch as a reviewed pull request. The agent runs every available gate locally first, refuses to push if anything is red, then commits + pushes + opens a PR.

## Refuse to ship if

- **On master/main.** Never push directly. Surface the error, ask the user to branch first.
- **No diff vs base branch.** Nothing to ship; tell the user there are no changes.
- **Suspicious files in the diff** — `.env*`, `*.pem`, files matching `*credentials*`, files outside the repo root. List them and require explicit confirmation before staging.
- **Detached HEAD.** Branch first.

## Gate discovery (project-agnostic)

The skill is universal. It does NOT hardcode a project's gates. Discovery rules:

1. **Single-package repo** (root `package.json` exists): run every script that matches one of:
   - `lint`, `lint:*`
   - `typecheck`, `typecheck:*`, `type-check`
   - `test`, `test:*` (excluding `test:watch`, `test:debug`, `test:dev`)
   - `build`, `build:*`
   - `e2e`, `test:e2e`
2. **Monorepo** (workspaces or `packages/*/package.json`): run the matching scripts in every workspace package that defines them. Workspace order doesn't matter as long as each runs to completion.
3. **Override file** `.claude/ship.yml` in the repo root, if present, takes precedence:
   ```yaml
   gates:
     - cwd: packages/api
       cmd: bun run typecheck
     - cwd: packages/dashboard
       cmd: bun run test
   skip:
     - test:watch
   ```
4. **Non-Node repos**: fall back to common entry points if the user has a `Makefile` (`make test`, `make lint`), `justfile`, or language-specific config (`cargo test`, `pytest`). If none detected, ask the user how to test instead of pushing without verification.

Run gates in parallel where possible. Stream output. **If any gate fails, stop. Print which one and the tail of its output. Do not stage, commit, or push.** Do not "fix and retry" silently — the failure may be intentional WIP the user wants to see.

## Pre-commit hygiene

- **Stage explicitly.** Never `git add -A` or `git add .`. List `git status` modified+untracked files; exclude obvious leakers (`.env*`, build artifacts, `node_modules/`, `dist/`, OS files); confirm anything unexpected.
- **Read each file once before committing** to make sure no half-finished work is being shipped silently.
- **Commit message.** Imperative subject under 72 chars. Body explains the *why*, not the *what* — the diff already shows the what. Pass via HEREDOC to preserve formatting. Always include `Co-Authored-By: Claude Opus 4.7 (1M context) <noreply@anthropic.com>` as the trailer if the work was paired with the agent.

## Push + PR

1. `git push -u origin <branch>`. Never `--force`, never `--no-verify`.
2. `gh pr create --title <subject> --body <generated>`.
3. **Generate the PR body** from `git log <base>..HEAD --reverse` on the branch:
   - **Summary**: 1–3 bullets synthesized from the commit subjects (not just dumped — group related commits).
   - **Test plan**: a checklist of the actual things to verify based on what was touched (UI changes → "click through the page", schema changes → "migrate up/down", etc.). Don't write generic "tests pass" — the gates already ran.
4. Print the resulting PR URL.

## Hard "no"s

- **No `--force` push.** If the branch has diverged, surface it and ask.
- **No `--no-verify` / `--no-gpg-sign`.** If a hook is failing, fix the underlying cause.
- **No amending a commit that's been pushed.** Create a new commit instead.
- **No squashing without explicit user direction.** The branch's commit history is information.
- **No silent workspace mutations** (changing `.gitconfig`, branch protection, default branch, etc.).

## Reusing this skill

When you see this skill match the user's intent, follow the steps above in order. If the user says "ship without tests" or similar, **ask before bypassing** — gates exist to prevent shipping broken code; bypassing them is a deliberate decision, not a default.
