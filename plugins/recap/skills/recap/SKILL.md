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
