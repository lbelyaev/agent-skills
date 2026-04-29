# git-flow

Procedural git workflow skills.

## Skills

- **`ship`** — land the current branch as a PR. Runs every available gate locally first; refuses to push if anything fails. Stages explicitly, commits with a meaningful message, pushes (no `--force`, no `--no-verify`), opens a PR with a body derived from the branch's commit history. Project-agnostic — discovers gates from `package.json` scripts (or `.claude/ship.yml` override).

## Per-repo override

Drop `.claude/ship.yml` in your repo to override gate discovery:

```yaml
gates:
  - cwd: packages/api
    cmd: bun run typecheck
  - cwd: packages/dashboard
    cmd: bun run test
skip:
  - test:watch
```
