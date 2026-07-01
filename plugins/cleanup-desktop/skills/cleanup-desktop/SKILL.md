---
description: Organize macOS Desktop files into categorized folders. Sorts screenshots by year-month, PDFs, archives, videos, and images. Triggered by "clean up desktop", "organize desktop", "sort screenshots".
---

# cleanup-desktop スキル

デスクトップ（`~/Desktop/`）のファイルをフォルダ別に自動整理する。

## 整理ルール

| 対象ファイル | 移動先 |
|---|---|
| `スクリーンショット *.png` / `Screenshot *.png` | `~/Desktop/screenshots/YYYY-MM/` |
| `*.pdf` | `~/Desktop/docs/` |
| `*.zip` / `*.7z` / `*.tar.gz` | `~/Desktop/archives/` |
| `*.mp4` / `*.mov` / `*.avi` | `~/Desktop/videos/` |
| `*.jpg` / `*.jpeg` / `*.png`（スクショ以外） | `~/Desktop/images/` |
| その他のファイル | `~/Desktop/misc/` |

## 手順

### Step 1: 現状確認

```bash
ls -la ~/Desktop/
```

ファイル一覧を確認し、上記の整理ルールに従って「何がどこに移動されるか」の一覧をユーザーに提示する。
**Step 2以降（ディレクトリ作成・実際の移動）は、ユーザーが明示的に承認してから実行する。** 削除は行わないが、移動も無断で進めない。

### Step 2: 移動先ディレクトリ作成（承認後に実行）

今月のスクリーンショットフォルダを含む必要なディレクトリを作成する。

```bash
MONTH=$(date +"%Y-%m")
mkdir -p ~/Desktop/screenshots/$MONTH
mkdir -p ~/Desktop/docs
mkdir -p ~/Desktop/archives
mkdir -p ~/Desktop/videos
mkdir -p ~/Desktop/images
mkdir -p ~/Desktop/misc
```

### Step 3: スクリーンショットの仕分け

ファイル名の日付から適切なYYYY-MMフォルダへ振り分ける。

```bash
# スクリーンショットを年月別フォルダへ
for f in ~/Desktop/スクリーンショット\ *.png ~/Desktop/Screenshot\ *.png; do
  [ -f "$f" ] || continue
  # ファイル名から年月を抽出（例: スクリーンショット 2026-03-04 → 2026-03）
  ym=$(echo "$f" | grep -oE '[0-9]{4}-[0-9]{2}' | head -1)
  if [ -n "$ym" ]; then
    mkdir -p ~/Desktop/screenshots/$ym
    mv "$f" ~/Desktop/screenshots/$ym/
  else
    mv "$f" ~/Desktop/screenshots/$(date +"%Y-%m")/
  fi
done
```

### Step 4: その他ファイルの仕分け

```bash
# PDF
for f in ~/Desktop/*.pdf; do [ -f "$f" ] && mv "$f" ~/Desktop/docs/; done
# アーカイブ
for f in ~/Desktop/*.zip ~/Desktop/*.7z ~/Desktop/*.tar.gz ~/Desktop/*.dmg; do [ -f "$f" ] && mv "$f" ~/Desktop/archives/; done
# 動画
for f in ~/Desktop/*.mp4 ~/Desktop/*.mov ~/Desktop/*.avi; do [ -f "$f" ] && mv "$f" ~/Desktop/videos/; done
# 画像（スクショ以外）
for f in ~/Desktop/*.jpg ~/Desktop/*.jpeg ~/Desktop/*.png ~/Desktop/*.gif ~/Desktop/*.webp; do [ -f "$f" ] && mv "$f" ~/Desktop/images/; done
```

### Step 5: 結果確認

```bash
ls -la ~/Desktop/
echo "--- スクリーンショット ---"
ls ~/Desktop/screenshots/ 2>/dev/null
```

## 注意事項

- **削除は行わない**。仕分けのみ。不要ファイルの削除はユーザーが手動で判断する
- `.DS_Store` / `.localized` は触らない
- `iCloud Drive` 同期中のファイルは移動前に同期完了を確認する
- フォルダ自体（`screenshots/`, `docs/` 等）が既に存在する場合はそのまま利用する

## Gotchas

**iCloud Drive同期中のファイルを移動すると同期が壊れる**
同期未完了のファイルは `.icloud` という隠し拡張子の部分ファイル（例: `.本体名.拡張子.icloud`）として存在する。
移動前に以下で同期状況を確認する:
```bash
brctl status ~/Desktop 2>/dev/null | head -20
ls -la ~/Desktop/ | grep '\.icloud$'
```
`.icloud`拡張子のファイルが残っている場合はダウンロード未完了なので、そのファイルは仕分け対象から除外し、ユーザーに「同期完了後に再実行してください」と伝える。

**ユーザー承認前にmkdir/mvを実行しない**
Step 1の一覧提示は「提案」であり実行ではない。ユーザーの明示的なOKが出るまでStep 2以降のコマンドを実行してはならない。
