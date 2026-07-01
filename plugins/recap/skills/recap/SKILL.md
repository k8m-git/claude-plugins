---
description: Display a summary of the current conversation context. Use after /compact or when resuming work. Triggered by "recap", "what was I doing", "context summary", "summarize work".
---

# Recap — コンテキスト概要表示

compact 後や作業再開時に、今の状況を整理して表示する。

## 手順

以下のフォーマットで現在のコンテキストをまとめて出力する：

---

## 📋 作業コンテキスト

### プロジェクト
- 作業中のプロジェクト・ディレクトリ

### 関連イシュー
- Linear/GitHub イシューのタイトルとURL（あれば）

### やっていたこと
- 直前の作業内容（機能追加・バグ修正・調査など）

### 進捗・現在地
- どこまで完了しているか
- 未完了のタスク

### 次のアクション
- 再開すべき具体的なステップ

### 重要なメモ
- 忘れてはいけない制約・決定事項・注意点

---

コンテキストが不明な部分は「不明」と記載し、ユーザーに確認を促す。

## 実行後の記録

出力後、以下で `~/.claude/recap-log.json` に実行履歴を追記する（次回 `/recap` 実行時に前回の「次のアクション」を参照できるようにするため）。

```bash
python3 -c "
import json, os
from datetime import datetime

log_path = os.path.expanduser('~/.claude/recap-log.json')
try:
    with open(log_path) as f:
        data = json.load(f)
except (FileNotFoundError, json.JSONDecodeError):
    data = []

# PROJECT, NEXT_ACTION は実行時に埋め込む
data.append({
    'at': datetime.now().isoformat(timespec='seconds'),
    'project': 'PROJECT',
    'next_action': 'NEXT_ACTION',
})
data = data[-20:]  # 直近20件のみ保持

with open(log_path, 'w') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
" 2>/dev/null || true
```

## Gotchas

**compact直後はコンテキストが要約されている**
compactによって詳細が失われている場合がある。
不明な項目は推測せず「不明（compactで省略された可能性あり）」と明記する。

**作業が複数プロジェクトにまたがる場合**
プロジェクトが複数ある場合はそれぞれ分けて記載する。
1つにまとめると「次のアクション」が曖昧になるため。

**「次のアクション」が不明な場合は確認する**
コンテキストから次のステップが読み取れない場合は推測で埋めず、
「次のアクションが不明です。何から再開しますか？」とユーザーに聞く。

**要約前に必ず `git status` / `git log -3` で現在地を確認する**
会話履歴の記憶だけで「完了した」と書くと、実際はuncommittedのまま放置されているケースがある。
出力前に一度 `git status` を実行し、変更ファイルの有無を「進捗・現在地」に反映する。

**`~/.claude/recap-log.json` に前回の記録がある場合は参照する**
存在すれば直近1〜2件を読み、「前回の次のアクション」が今回のコンテキストと矛盾しないか確認してから出力する。矛盾がある場合はユーザーに指摘する。
