---
description: >
  Review the Claude Code workspace structure. Detects drift between CLAUDE.md documentation and actual
  files, checks skill/agent alignment, hooks, rules, and notes.
  Triggered by "workspace review", "review CLAUDE.md", "audit workspace", "check workspace config",
  "clean up CLAUDE.md", "workspace audit".
---

# Workspace Review — Claude Code Workspace Audit

Reviews the current workspace's Claude Code configuration and detects misalignments between
documentation and reality.

**For skill quality auditing, use `workspace-skill-review` instead.**
This skill focuses on the workspace skeleton: CLAUDE.md, rules, hooks, directory structure.

Reference:
- https://claude.com/blog/steering-claude-code-skills-hooks-rules-subagents-and-more — 7-layer design guide for Skills/Hooks/Rules/Subagents
- https://code.claude.com/docs/en/best-practices#write-an-effective-claude-md — What to include/exclude in CLAUDE.md

---

## Review Targets

Detect the workspace root automatically:

```bash
# Find the workspace root (directory containing CLAUDE.md)
WORKSPACE=$(pwd)
# Walk up to find CLAUDE.md if not in current directory
while [ "$WORKSPACE" != "/" ] && [ ! -f "$WORKSPACE/CLAUDE.md" ]; do
  WORKSPACE=$(dirname "$WORKSPACE")
done
echo "Workspace root: $WORKSPACE"
```

| Target | Path (relative to workspace root) |
|---|---|
| Project constitution | `CLAUDE.md`, `CLAUDE.local.md` |
| Global config | `~/.claude/CLAUDE.md` |
| Rules | `.claude/rules/*.md` |
| Skills (listing only) | `.claude/skills/*/SKILL.md` (frontmatter only) |
| Agents (listing only) | `.claude/agents/*.md` (frontmatter only) |
| Hook config | `.claude/settings.json`, `.claude/settings.local.json` |
| Directory structure | workspace root top-level |
| Notes / docs | `notes/*.md` (listing only, if present) |

---

## Procedure

### Phase 1: Collect Configuration Info

Run in parallel with Read / Bash:

```bash
# Detect workspace root
WORKSPACE=$(pwd)
while [ "$WORKSPACE" != "/" ] && [ ! -f "$WORKSPACE/CLAUDE.md" ]; do
  WORKSPACE=$(dirname "$WORKSPACE")
done

ls -la "$WORKSPACE/"
ls "$WORKSPACE/.claude/skills/" 2>/dev/null || echo "no skills/"
ls "$WORKSPACE/.claude/agents/" 2>/dev/null || echo "no agents/"
ls "$WORKSPACE/.claude/rules/"  2>/dev/null || echo "no rules/"
ls "$WORKSPACE/notes/"          2>/dev/null | head -20 || echo "no notes/"
```

Read these files:
- `CLAUDE.md`
- `CLAUDE.local.md` (if present)
- `.claude/settings.json` (if present)
- `.claude/settings.local.json` (if present)
- all files under `.claude/rules/`

### Phase 2: CLAUDE.md Quality Check

Evaluate against official best practices.

**Include (✅) vs Remove (❌):**

| Verdict | Criterion |
|---|---|
| ✅ Keep | Bash commands Claude can't infer from code |
| ✅ Keep | Non-default code style rules |
| ✅ Keep | Test procedures and test runner setup |
| ✅ Keep | Branch naming / PR conventions |
| ✅ Keep | Project-specific architecture decisions |
| ✅ Keep | Dev environment quirks (required env vars, etc.) |
| ✅ Keep | Non-obvious gotchas and pitfalls |
| ❌ Cut | Anything readable from the code itself |
| ❌ Cut | Standard language conventions Claude already knows |
| ❌ Cut | Detailed API docs (a link is enough) |
| ❌ Cut | Frequently-changing information |
| ❌ Cut | Long explanations or tutorials |
| ❌ Cut | File-by-file codebase descriptions |
| ❌ Cut | Obvious content like "write clean code" |

**Checklist:**
- [ ] For every line: "Would Claude make a mistake if this were removed?"
- [ ] Under 200 lines? (Important rules get buried when longer)
- [ ] Any "always do X" instructions? (→ move to hooks)
- [ ] Any "never do X" instructions? (→ move to hooks/permissions)
- [ ] Any procedure over 30 lines? (→ move to a skill)
- [ ] Heavy content that could be split with `@path/to/file` imports?
- [ ] Path-specific rules? (→ move to `.claude/rules/` with `paths:` scope)

### Phase 3: Configuration Alignment Check

**Verify CLAUDE.md descriptions match actual files:**

1. Skills listed in CLAUDE.md actually exist in `.claude/skills/`?
2. Agents listed in CLAUDE.md actually exist in `.claude/agents/`?
3. External files referenced with `@path/to/file` actually exist?
4. "Directory Structure" section in CLAUDE.md matches actual workspace layout?
5. Archived/deprecated projects still mentioned in CLAUDE.md?

**Hook config check:**
- Scripts/commands referenced by hooks actually exist?
- Any hooks that are no longer needed?

**Output styles check:**
- Check whether `.claude/output-styles/` exists.
- If unused, optionally suggest the built-in styles (Proactive / Explanatory / Learning) — not required.

**Leftover worktrees:**
```bash
find "$WORKSPACE" -maxdepth 3 -type d -name "*.worktrees" 2>/dev/null
```
Check whether stale branches (likely already merged/finished) remain under `*.worktrees/`.
When in doubt, just list them — don't suggest deletion.

### Phase 4: Rules File Check

For each file under `.claude/rules/`:
- Is `paths:` scope set appropriately?
- Any content that should move from CLAUDE.md to rules?
- Any content better handled by hooks than rules?

### Phase 5: Notes / Docs Inventory

Check the `notes/` directory (if present):
- Files referenced in CLAUDE.md that don't exist?
- Files potentially stale (last modified 3+ months ago)?
- Notes containing important info that should be added to CLAUDE.md?

### Phase 6: Report Output

```markdown
## Workspace Review Report

**Reviewed at**: YYYY-MM-DD HH:MM
**Workspace root**: /path/to/workspace

---

### 🔴 Action Required (drift detected)
- Listed in CLAUDE.md but missing: `skills/xxx`
- Referenced file not found: `@notes/xxx.md`

### 🟡 Recommended Improvements
- "Always do X" instruction in CLAUDE.md → consider moving to a hook
- Procedure (N lines) inside CLAUDE.md → consider extracting to a skill: `{title}`
- CLAUDE.md is {N} lines → deletion candidates: `{specific lines}`

### 🟢 Looks Good
- Rule files have appropriate paths: scopes
- Skill listing matches actual files

---

### Recommended Actions (priority order)
1. {Top priority: drift between docs and reality}
2. {Next: content to extract from CLAUDE.md}
3. {Optional: cleanup and inventory}
```

### Phase 7: Apply Fixes (after user approval)

Edit only what the user explicitly asks to fix after seeing the report.
No bulk changes. Show the proposed change before applying it.

---

## Gotchas

**Long CLAUDE.md isn't always a problem**
Length matters less than whether Claude already knows the content.
Always explain *why* something can be removed when suggesting deletions.

**Symlinked skills/agents may live elsewhere**
Symlinks under `.claude/skills/` pointing to `../plugins/` are plugin-managed.
Note "managed by plugin" rather than suggesting edits.

**CLAUDE.local.md is gitignored personal config**
Overlaps with CLAUDE.md may be intentional personal overrides.
Ask "is this intentional?" before proposing changes.

**Two hook config files: settings.json and settings.local.json**
Check both if `settings.local.json` exists — personal hooks may be defined there.

**"Always do X" → hook migration requires care**
Hooks are code and have maintenance cost.
Low-frequency or conditional instructions may be better left in CLAUDE.md or a skill.

**CLAUDE.md not found when starting from an unrelated directory**
The walk-up loop may reach `/` without finding CLAUDE.md.
If `$WORKSPACE` becomes `/`, ask the user to specify the project root manually.

**Don't flag hardcoded org-specific values as "should be generalized"**
Skills with hardcoded Slack channel IDs, Linear org names, etc. are intentional.
Mark them as "org-specific skill — appropriate as-is" rather than suggesting generalization.

**Symlinked skills (scan, similar, etc.) are plugin-managed**
Symlinks under `.claude/skills/` pointing to `../plugins/` are managed by a plugin.
Note "plugin-managed — edits belong in the plugin source" rather than editing directly.
