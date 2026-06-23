---
description: >
  Audit Claude Code skills against 8 quality axes based on Anthropic's skill design principles.
  Scores each skill and surfaces top improvement candidates.
  Triggered by "review my skills", "skill review", "audit skills", "skill quality check",
  "check skill quality", "improve my skills".
---

# Workspace Skill Review — Skill Quality Audit

Evaluates global and project skills against 8 axes derived from Anthropic's skill design principles,
identifies the highest-priority improvements, and applies fixes one at a time with user approval.

Reference:
- https://claude.com/blog/lessons-from-building-claude-code-how-we-use-skills — 9 categories & 8 principles (basis for the scoring axes)

---

## Scoring Axes (from Anthropic blog principles)

| # | Axis | What to evaluate |
|---|---|---|
| 1 | **Description has trigger words** | Contains natural-language keywords so the model can auto-invoke it |
| 2 | **Non-obvious content only** | Avoids telling Claude things it already knows; focuses on gotchas and constraints |
| 3 | **Gotchas section present** | Real failures, API quirks, and common errors are documented |
| 4 | **Progressive disclosure** | Large skills use subfolder references instead of one long file |
| 5 | **Skill memory** | Execution results / config are persisted to a JSON log for future use |
| 6 | **Not over-railroaded** | Leaves Claude room to make situational judgments |
| 7 | **Scripts / tools provided** | Boilerplate code or reusable helpers are included |
| 8 | **Clear 9-category membership** | Belongs recognizably to one of the 9 categories below |

**9 categories reference**:
Library & API Reference / Product Verification / Data Fetching & Analysis /
Business Process Automation / Code Scaffolding / Code Quality & Review /
CI/CD & Deployment / Runbooks / Infrastructure Operations

---

## Procedure

### Phase 1: Collect Skill List

Detect the workspace root automatically:

```bash
WORKSPACE=$(pwd)
while [ "$WORKSPACE" != "/" ] && [ ! -f "$WORKSPACE/CLAUDE.md" ]; do
  WORKSPACE=$(dirname "$WORKSPACE")
done
echo "Workspace: $WORKSPACE"
ls ~/.claude/skills/       2>/dev/null || echo "no global skills"
ls "$WORKSPACE/.claude/skills/" 2>/dev/null || echo "no project skills"
```

Read each `SKILL.md` found. If there are more than 10 skills, use an **Explore agent for parallel
loading** to avoid excessive context consumption — have it return only the scores.

### Phase 2: Score Each Skill (8 axes)

Rate each axis with:

| Symbol | Meaning |
|---|---|
| ◎ | Clearly implemented |
| ○ | Partially present |
| △ | Insufficient / room to improve |
| ✕ | Completely absent |

Output a scoring table:

```
| Skill | ①desc | ②non-obvious | ③Gotchas | ④PD | ⑤memory | ⑥flex | ⑦script | ⑧category | Total |
|-------|------|-------------|---------|-----|---------|------|---------|-----------|-------|
| recap | ◎ | ○ | △ | ○ | ✕ | ◎ | ✕ | ◎ | 22/32 |
...
```

**Score calculation**: ◎=4, ○=3, △=2, ✕=1. Max 32 points.

**Progressive disclosure判定 (axis ④):**
- Subfolder contains files beyond `SKILL.md` → ◎
- 50 lines or fewer → flat is fine (○)
- 50–150 lines → consider splitting (△)
- Over 150 lines → should be split (✕)

**Skill memory判定 (axis ⑤):**
- Writes to a persistent file (JSON log, etc.) → ◎
- Only in-conversation memory → △
- No memory at all → ✕

### Phase 3: Priority Report

Present the bottom 3–5 skills as "improvement candidates". For each:
- **What's missing** (specific axes)
- **Example fix** (rewritten description, sample Gotchas entry, etc.)

Priority order:
1. ③ Gotchas absent (✕) on an actively used skill → highest priority
2. ① No trigger words in description → high risk of never being found
3. ④ No progressive disclosure AND over 150 lines → context waste

### Phase 4: Apply Fixes (after user approval)

Edit only the skills the user explicitly approves. Fix one skill at a time — always confirm before
moving to the next.

### Phase 5: Log Results to work-log.json

```bash
python3 -c "
import json, os
from datetime import date, datetime

log_path = os.path.expanduser('~/.claude/work-log.json')
today = date.today().isoformat()

try:
    with open(log_path) as f:
        data = json.load(f)
except (FileNotFoundError, json.JSONDecodeError):
    data = {}

entry = data.get(today, {'date': today})
# Replace SCORES_JSON and TOP_ISSUES_JSON at runtime
entry['skill_review'] = {
    'reviewed_at': datetime.now().strftime('%H:%M'),
    'scores': SCORES_JSON,
    'top_issues': TOP_ISSUES_JSON,
}
data[today] = entry

with open(log_path, 'w') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
print('Saved skill_review to work-log.json')
" 2>/dev/null || true
```

- `SCORES_JSON`: `{"recap": 22, "git-worktree": 28, ...}`
- `TOP_ISSUES_JSON`: `[{"skill": "recap", "issues": ["③ Gotchas absent", "⑤ no memory"]}]`

---

## Gotchas

**Too many skills → context overload**
Reading all skills in one conversation can consume a large number of tokens.
Check the skill count in Phase 1; if over 10, delegate to an Explore agent to load and score
them, returning only the scores — not the full content.

**Symlinked / plugin-managed skills**
Skills under `.claude/skills/` that symlink to `../plugins/` are managed by a plugin.
Score them but note "plugin-managed — edits belong in the plugin source" rather than editing
directly.

**Skill memory false positives**
Distinguish between "writes to a persistent file" (◎) and "only notes things in the
conversation" (△). Only persistent file writes qualify as real skill memory.

**Over-railroading is subjective**
Many steps ≠ over-railroading. The question is whether Claude is given *no room to judge*.
If the skill says "always output in exactly this format" repeatedly → △.
If Claude can adapt based on context → ○.
