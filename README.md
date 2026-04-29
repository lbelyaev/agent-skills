# lbelyaev/agent-skills

A personal Claude Code plugin marketplace.

## Install

```bash
claude plugin marketplace add lbelyaev/agent-skills
claude plugin install git-flow@lbelyaev-agent-skills
```

## Plugins

- **[git-flow](plugins/git-flow)** — git workflow skills (`ship`, etc.). Lands a branch as a PR with all gates green; refuses to push on red.

## Layout

```
.claude-plugin/marketplace.json   # marketplace manifest
plugins/<name>/
  .claude-plugin/plugin.json      # plugin metadata
  README.md
  skills/<skill>/SKILL.md         # individual skill
```
