---
name: save-tokens
description: セッションのトークン消費を半減させる省トークンモードを有効化する。stats-cache.jsonから使用パターンを解析し、最適な削減戦略を適用する。「トークン節約」「save tokens」「省トークン」と言ったときにも使用。
argument-hint: "[aggressive|status]"
allowed-tools: Bash
---

以下の手順で省トークンモードを即座に有効化し、このセッションの残りすべてに適用せよ。

## モード判定

- `$ARGUMENTS` が空 または `standard`: 標準モードを有効化
- `$ARGUMENTS` が `aggressive`: 強化モードを有効化（標準モード＋追加制約）
- `$ARGUMENTS` が `status`: 現在のモード状態を1行で報告して終了

## ステップ1: 使用パターンの解析（status以外）

Bash で以下のコマンドを実行し、stats-cache.json からトークン使用パターンを取得せよ:

```bash
python3 -c "
import json, sys
import os; path = os.path.expanduser('~/.claude/stats-cache.json')
if not os.path.exists(path):
    print('cache_hit=0.0')
    print('out_in_ratio=0.0')
    print('recent_7d=0')
    print('out_tokens=0')
    print('in_tokens=0')
    print('no_data=true')
    sys.exit(0)
with open(path) as f:
    d = json.load(f)

usage = d.get('modelUsage', {})
total = {'in': 0, 'out': 0, 'cache_read': 0, 'cache_create': 0}
for m, v in usage.items():
    if v.get('cacheReadInputTokens', 0) > 0 or v.get('outputTokens', 0) > 0:
        total['in'] += v.get('inputTokens', 0)
        total['out'] += v.get('outputTokens', 0)
        total['cache_read'] += v.get('cacheReadInputTokens', 0)
        total['cache_create'] += v.get('cacheCreationInputTokens', 0)

all_input = total['in'] + total['cache_read'] + total['cache_create']
cache_hit = total['cache_read'] / all_input * 100 if all_input else 0
out_in_ratio = total['out'] / total['in'] * 100 if total['in'] else 0

# 最近7日分のトークン
daily = d.get('dailyModelTokens', [])[-7:]
recent_total = sum(sum(day.get('tokensByModel', {}).values()) for day in daily)

print(f'cache_hit={cache_hit:.1f}')
print(f'out_in_ratio={out_in_ratio:.1f}')
print(f'recent_7d={recent_total}')
print(f'out_tokens={total[\"out\"]}')
print(f'in_tokens={total[\"in\"]}')
"
```

出力から以下を読み取る:
- `cache_hit`: キャッシュヒット率（%）
- `out_in_ratio`: 出力/入力比（%）
- `recent_7d`: 直近7日間の総トークン数

## ステップ2: ボトルネック診断と優先ルール決定

ステップ1の出力に `no_data=true` が含まれる場合:
- 「stats-cache.json未存在。デフォルトルールで省トークンモード有効化。」と報告
- 全カテゴリ均等適用でステップ3へ進む

それ以外は読み取った値を以下の基準で判定し、適用するルールカテゴリの優先度を決定する:

| 条件 | 判定 | 重点カテゴリ |
|------|------|-------------|
| out_in_ratio > 50% | 出力過多 | **カテゴリ1を最優先** |
| cache_hit < 80% | キャッシュ低効率 | **カテゴリ5を強化** |
| recent_7d が非常に大きい | 使用量が多い | **カテゴリ3・4を強化** |

診断結果を2行以内でユーザーに伝える（数値と判定理由を含める）。

## ステップ3: 省トークンモードの有効化

以下のルールをセッション残り全体に即座に適用すると宣言し、実行開始せよ。

---

### カテゴリ1: 出力トークン削減（最重要: 単価が入力の5倍・キャッシュ読みの50倍）

1. 前置きは最大1文。「承知しました」等の定型句を禁止
2. 末尾のまとめ・クロージング禁止
3. 単一セクションの回答にMarkdown見出しを使わない
4. コードブロック: 変更行＋前後3行のみ。ファイル全体の再掲禁止
5. 3項目以下のリストはインライン形式
6. ファイル編集後のbefore/after散文禁止
7. ツール実行後はタスク関連の結果のみ報告
8. ファイル読み取り後、内容をエコーバックしない
9. 「why」は非自明な場合のみ説明
10. クロージング禁止

### カテゴリ2: 強化モード追加ルール（aggressive時のみ）

11. 前置きゼロ。行動から直接開始
12. 連続するツール呼び出し間の散文禁止
13. ファイル編集の確認メッセージ不要、即実行
14. Glob/Grep結果のパス一覧省略（件数が想定外の場合を除く）

### カテゴリ3: ツール使用最適化

- `Bash(find/ls)` → `Glob`
- `Bash(grep/rg)` → `Grep`
- `Bash(cat/head/tail)` → `Read`（offset/limit指定）
- 部分読み取り: 対象位置が既知なら該当部分のみ読む
- 並列実行: 独立した情報取得は1レスポンスにまとめて並列呼び出し
- 再読み禁止: 既読かつ未変更のファイルを再度読まない
- 単純タスクは `Agent`（model: haiku）に委譲

### カテゴリ4: コンテキスト管理

- コンテキストが長い場合、次タスク前に `/compact` を提案
- タスク完了後の再要約禁止
- 新規ファイル作成後の全内容エコー禁止

### カテゴリ5: キャッシュ最適化

- 同一ファイルへの複数編集は1回の `Edit` にまとめる
- 会話の一貫性を保ちキャッシュヒット率を維持（5分TTL）

---

## 注意

- `/compact` 実行後はモードが失われる場合がある → 再実行で復元可能（べき等）
- 品質（正確性・完全性）は維持すること。冗長性のみを排除する
