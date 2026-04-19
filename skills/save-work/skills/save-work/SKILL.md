---
name: save-work
description: /clearの前に作業内容を保存したいとき、「作業を保存」「save work」「clearの前に保存」「セッションを保存」と言ったときに使用する。現在の会話の作業内容をMarkdownファイルに保存する。
allowed-tools: Agent, Bash, Write
model: haiku
---

現在の会話における作業内容を以下の手順で保存してください。

## 保存先
カレントディレクトリ配下の `.claude-session/` フォルダ（`$ARGUMENTS` が指定された場合はその配下）にタイムスタンプ付きのファイル名で保存する。
- ファイル名形式: `作業メモ_YYYY-MM-DD_HH-MM-SS.md`

## 保存内容

以下の構成でMarkdownファイルを作成してください：

```markdown
# 作業セッション: YYYY-MM-DD HH:MM

## 作業概要
（このセッションで何をしたかを2〜3文で要約）

## 実施した作業
（箇条書きで具体的な作業内容を列挙）

## 主な変更ファイル
（編集・作成・削除したファイルのパス一覧）

## 未完了・次のアクション
（まだ残っている作業や次にやること）

## 重要なメモ
（判断の背景、注意点、特記事項など）
```

## 手順

1. 現在の会話内容から作業メモの内容を整理・作成する
2. `Agent` ツールで **model: haiku** を使い、以下を実行させる:
   - `Bash` で現在時刻を取得: `date +%Y-%m-%d_%H-%M-%S`
   - `Bash` でカレントディレクトリを確認: `pwd`
   - `Bash` で `.claude-session` ディレクトリを作成: `mkdir -p <cwd>/.claude-session`
   - `Write` ツールで `<cwd>/.claude-session/作業メモ_<timestamp>.md` に書き込む（絶対パスを使用）
3. 保存完了をユーザーに伝え、ファイルパスを表示する
