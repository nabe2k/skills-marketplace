# skills-marketplace

Claude Code カスタムスキルのマーケットプレイス。

## スキル一覧

| スキル名 | 説明 | タグ |
|----------|------|------|
| [session-memo](skills/session-memo/skills/session-memo/SKILL.md) | セッションの作業メモをMarkdownに保存 | session, productivity |
| [save-tokens](skills/save-tokens/skills/save-tokens/SKILL.md) | 省トークンモードを有効化してコストを削減 | cost, productivity |

## インストール方法

### プラグインとしてインストール（推奨）

Claude Code のプラグインシステムを使用する：

```
# マーケットプレイスを追加（初回のみ）
/plugin marketplace add nabe2k/skills-marketplace

# スキルを個別にインストール
/plugin install session-memo@skills-marketplace
/plugin install save-tokens@skills-marketplace
```

### 手動インストール

スキルファイルを直接配置する場合：

```bash
# グローバル（全プロジェクト共通）
mkdir -p ~/.claude/skills/<skill-name>
cp skills/<skill-name>/skills/<skill-name>/SKILL.md ~/.claude/skills/<skill-name>/SKILL.md

# プロジェクトローカル（特定プロジェクト専用）
mkdir -p <project-root>/.claude/skills/<skill-name>
cp skills/<skill-name>/skills/<skill-name>/SKILL.md <project-root>/.claude/skills/<skill-name>/SKILL.md
```

配置するだけで Claude Code が自動検出し、`/<skill-name>` コマンドとして利用可能になる。

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

## マイグレーションガイド

### v2.0.0: `save-work` → `session-memo` (破壊的変更)

`save-work` スキルは `session-memo` に改名されました。

**移行手順:**

```bash
# 旧スキルをアンインストール
/plugin uninstall save-work@skills-marketplace

# 新スキルをインストール
/plugin install session-memo@skills-marketplace
```

手動インストールの場合:
```bash
# 旧ディレクトリを削除
rm -rf ~/.claude/skills/save-work

# 新スキルを配置
mkdir -p ~/.claude/skills/session-memo
cp skills/session-memo/skills/session-memo/SKILL.md ~/.claude/skills/session-memo/SKILL.md
```

## 新規スキルの追加

1. `skills/<skill-name>/skills/<skill-name>/SKILL.md` を作成
2. `skills/<skill-name>/.claude-plugin/plugin.json` を作成
3. `.claude-plugin/marketplace.json` の `plugins` 配列にエントリを追加
4. `catalog.json` にエントリを追加
5. Pull Request を送る
