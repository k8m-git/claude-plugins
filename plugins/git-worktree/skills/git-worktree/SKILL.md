---
description: Create a git worktree for a branch to enable parallel development, and clean up finished worktrees. Triggered by phrases like "create a worktree for branch", "add worktree", "work in parallel on branch", "clean up worktrees", "list worktrees".
---

# Git Worktree Branch Creation

git worktreeを使い、ブランチごとに別ディレクトリで並列作業できるようにする。

## 作成手順

1. プロジェクトルートにいることを確認
2. **ブランチ命名規則の確認**: プロジェクトの`CLAUDE.md`にブランチ命名規則（例: `feature/{LINEAR-ID}-{説明}`）が定義されていればそれに従う。定義がなければデフォルトのケバブケース（`feature/add-login`等）を使う
3. worktree用ディレクトリを `../worktrees/<project>/<branch-name>` に作成
4. 以下のコマンドを実行:

```bash
# 新規ブランチを作成してworktreeを追加
git worktree add ../worktrees/<project>/<branch-name> -b <branch-name>

# 既存ブランチのworktreeを追加
git worktree add ../worktrees/<project>/<branch-name> <branch-name>
```

5. 実行後、以下で作成履歴を`~/.claude/git-worktree-log.json`に記録する:

```bash
python3 -c "
import json, os
from datetime import datetime

log_path = os.path.expanduser('~/.claude/git-worktree-log.json')
try:
    with open(log_path) as f:
        data = json.load(f)
except (FileNotFoundError, json.JSONDecodeError):
    data = []

# PROJECT, BRANCH, PATH は実行時に埋め込む
data.append({
    'project': 'PROJECT',
    'branch': 'BRANCH',
    'path': 'PATH',
    'created_at': datetime.now().isoformat(timespec='seconds'),
    'status': 'active',
})

with open(log_path, 'w') as f:
    json.dump(data, f, ensure_ascii=False, indent=2)
" 2>/dev/null || true
```

## クリーンアップ手順

「worktree片付けて」「worktree一覧見せて」等で起動する。

1. `git worktree list` で現在のworktree一覧を取得
2. `~/.claude/git-worktree-log.json` があれば参照し、各worktreeの作成日・ブランチ名を突き合わせる
3. 各ブランチのマージ済み判定を行う:
   ```bash
   git fetch origin
   git branch --merged main | grep -v '^\*\|  main\|  master'
   ```
   マージ済みと判定されたものには「マージ済み」ラベルを付けて一覧に表示する。
4. 一覧をユーザーに提示し、削除対象を選んでもらう（**削除の実行は必ずユーザー承認後**。マージ済みラベルは判断材料であり、自動削除の根拠にはしない。「マージ済みは全部消して」のような一括指示にはラベルを使って対応してよい）
5. 承認された対象について:
   ```bash
   git worktree remove <path>
   git branch -d <branch> 2>/dev/null || true
   ```
6. `git-worktree-log.json` の該当エントリを `status: "removed"` に更新する

## ルール

- worktreeのパスは `../worktrees/<プロジェクト名>/<ブランチ名>` に統一する
- ブランチ名の命名規則はプロジェクトの`CLAUDE.md`を優先し、なければケバブケース（例: `feature/add-login`, `fix/header-bug`）
- 作業完了後は上記クリーンアップ手順でユーザー承認を得てから削除する
- worktree一覧は `git worktree list` で確認できる

## トリガーフレーズ

以下のいずれかの表現でこのスキルを適用する:
- 「worktreeでブランチ切って」
- 「ワークツリーでブランチを切って」
- 「worktreeでブランチを作成して」
- 「ワークツリーでブランチを作成して」
- 「worktree片付けて」
- 「worktree一覧見せて」

## 使用例

### 作成時
ユーザーが作成系フレーズを使ったら:

1. ブランチ名を確認する
2. プロジェクト名を自動検出する
3. プロジェクトのCLAUDE.mdにブランチ命名規則があれば従う
4. worktreeを作成する
5. 作成履歴を記録する
6. 作成先のパスをユーザーに伝え、別ターミナルで開けることを案内する

### クリーンアップ時
ユーザーが片付け系フレーズを使ったら:

1. `git worktree list` と記録ログを突き合わせて一覧を提示する
2. ユーザーに削除対象を選んでもらう
3. 承認されたものだけ削除し、ログを更新する

## Gotchas

**同名ブランチのworktreeが既に存在する場合**
`git worktree add` は同じブランチを2つのworktreeで同時チェックアウトできない。
`git worktree list` で確認し、不要ならまず `git worktree remove <path>` してから再作成する。

**worktreeを削除する前にブランチを消してしまった場合**
`git branch -d <branch>` 後に `git worktree remove` しようとするとエラーになることがある。
`git worktree prune` を実行すると、対応するブランチが消えたworktreeの参照を一括クリーンアップできる。

**`../worktrees/` ディレクトリが存在しない場合**
`git worktree add` は中間ディレクトリを自動作成しない。
先に `mkdir -p ../worktrees/<project>` を実行してから worktree を追加する。

**メインブランチと同じブランチをworktreeで開こうとした場合**
現在チェックアウト中のブランチはworktreeに追加できない。
新規ブランチを切るか（`-b` オプション）、別のブランチ名を使う。

**未コミットの変更がある状態でworktreeを削除しようとした場合**
`git worktree remove` は未コミット変更があるとデフォルトで失敗する。
コミットまたはstashしてから削除するか、やむを得ない場合は `--force` を使う（変更は失われる）。

**`git worktree prune` と `git worktree remove` の違い**
- `remove <path>`: 特定のworktreeディレクトリとgitの参照を両方削除する
- `prune`: ディレクトリが既に手動削除されている場合に、残ったgit参照だけをクリーンアップする

**`git branch --merged` はsquash mergeを検出できない**
GitHubのPRをsquash mergeした場合、コミットハッシュが変わるため`git branch --merged main`ではマージ済みと判定されない。
「マージ済み」ラベルが付かないブランチでも、`gh pr list --state merged --head <branch>`でPRのマージ状態を確認し、マージ済みならユーザーに伝えた上で削除候補に含めてよい。
