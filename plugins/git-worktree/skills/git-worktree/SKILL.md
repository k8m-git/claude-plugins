---
description: Create a git worktree for a branch to enable parallel development. Triggered by phrases like "create a worktree for branch", "add worktree", "work in parallel on branch".
---

# Git Worktree Branch Creation

git worktreeを使い、ブランチごとに別ディレクトリで並列作業できるようにする。

## 手順

1. プロジェクトルートにいることを確認
2. worktree用ディレクトリを `../worktrees/<project>/<branch-name>` に作成
3. 以下のコマンドを実行:

```bash
# 新規ブランチを作成してworktreeを追加
git worktree add ../worktrees/<project>/<branch-name> -b <branch-name>

# 既存ブランチのworktreeを追加
git worktree add ../worktrees/<project>/<branch-name> <branch-name>
```

## ルール

- worktreeのパスは `../worktrees/<プロジェクト名>/<ブランチ名>` に統一する
- ブランチ名はケバブケースで命名（例: `feature/add-login`, `fix/header-bug`）
- 作業完了後は `git worktree remove <path>` でクリーンアップする
- worktree一覧は `git worktree list` で確認できる

## トリガーフレーズ

以下のいずれかの表現でこのスキルを適用する:
- 「worktreeでブランチ切って」
- 「ワークツリーでブランチを切って」
- 「worktreeでブランチを作成して」
- 「ワークツリーでブランチを作成して」

## 使用例

ユーザーが上記フレーズを使ったら:

1. ブランチ名を確認する
2. プロジェクト名を自動検出する
3. worktreeを作成する
4. 作成先のパスをユーザーに伝え、別ターミナルで開けることを案内する

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
