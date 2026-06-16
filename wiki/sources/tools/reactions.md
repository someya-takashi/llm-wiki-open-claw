---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/reactions
source_path: raw/docs/tools/reactions.md
doc_section: tools
title: "リアクション"
ingested: 2026-06-14
tags: [reactions, emoji, message-tool, channel]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/messages]]"
---

# リアクション（解説）

> 原典: `raw/docs/tools/reactions.md` ・ https://docs.openclaw.ai/ja-JP/tools/reactions

## 一言まとめ

エージェントが `message` ツールの `react` アクションで、メッセージに**絵文字リアクションを付与/削除**する。挙動はチャネルとトランスポートで異なる。

## 位置づけ

[[concepts/session-tool]] の `message` ツールの一機能で、配信は [[concepts/messages]] パイプライン。チャネルごとに対応状況が違う（チャネルページで言及）。

## 仕組み・ふるまい

- `message` ツール `react` でリアクション追加/削除。チャネル/トランスポートが対応していれば反映される。

## 設定・使い方の要点

- 軽量な「了解」「処理中」などの非言語フィードバックに使う。チャネル固有の制限あり。

## 用語と略称

- **リアクション（reaction）** = メッセージへの絵文字反応
- **`message` ツール** = 返信/チャネルアクションを送るツール
- **`react` アクション** = リアクションを操作する `message` のアクション

## 関連ページ

- [[concepts/session-tool]] / [[concepts/messages]]
- [[channels/zalouser]]
