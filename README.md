# skills-marketplace

Claude Code カスタムスキルのマーケットプレイス。

## スキル一覧

| スキル名 | 説明 | タグ |
|----------|------|------|
| [save-work](skills/save-work/SKILL.md) | /clearの前に作業内容をMarkdownに保存 | session, productivity |
| [save-tokens](skills/save-tokens/SKILL.md) | 省トークンモードを有効化してコストを削減 | cost, productivity |

## インストール方法

スキルをローカルの Claude Code 環境に追加する手順：

```bash
# スキルディレクトリに配置
cp skills/<skill-name>/SKILL.md ~/.claude/skills/<skill-name>/SKILL.md

# skill-registry.json の skill_dirs に以下が含まれていることを確認
# ~/.claude/skills
```

## スキルの構造

各スキルは `SKILL.md` 1ファイルで完結する。フロントマターで動作を制御する。

```markdown
---
name: skill-name
description: スキルの説明（トリガー判定に使用）
allowed-tools: Bash, Read, Write
model: haiku          # 省略時は現在のモデルを使用
---

（スキルの本文）
```

## 新規スキルの追加

1. `skills/<skill-name>/SKILL.md` を作成
2. `catalog.json` にエントリを追加
3. Pull Request を送る
