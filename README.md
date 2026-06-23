# claude-plugins

A collection of general-purpose Claude Code plugins by [k8m-git](https://github.com/k8m-git).

## Plugins

### git-worktree
Create git worktrees for parallel branch development. Includes a Gotchas section for common pitfalls.

**Skill**: `/git-worktree:git-worktree`

### recap
Display a structured summary of the current conversation context. Useful after `/compact` or when resuming work.

**Skill**: `/recap:recap`

### cleanup-desktop
Auto-organize macOS Desktop files into categorized subfolders (screenshots by year-month, PDFs, archives, videos, images).

**Skill**: `/cleanup-desktop:cleanup-desktop`

### workspace-review
Review your Claude Code workspace structure: CLAUDE.md quality, skill/agent alignment, hooks, and rules. Detects drift between documented and actual configuration.

**Skill**: `/workspace-review:workspace-review`

### workspace-skill-review
Audit your Claude Code skills against 8 quality axes (description triggers, Gotchas, progressive disclosure, skill memory, etc). Scores each skill and surfaces top improvement candidates.

**Skill**: `/workspace-skill-review:workspace-skill-review`

## Installation

```shell
# Add this marketplace
/plugin marketplace add k8m-git/claude-plugins

# Install individual plugins
/plugin install git-worktree@k8m-plugins
/plugin install recap@k8m-plugins
/plugin install cleanup-desktop@k8m-plugins
/plugin install workspace-review@k8m-plugins
/plugin install workspace-skill-review@k8m-plugins
```

## License

MIT
