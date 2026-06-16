---
type: concept
aliases: [Progress drafts, 進行中の下書き, progress mode]
tags: [progress-drafts, streaming, preview, tool-progress, ux]
related:
  - "[[concepts/streaming]]"
  - "[[concepts/messages]]"
sources:
  - "[[sources/concepts/progress-drafts]]"
updated: 2026-06-14
---

# 進行中の下書き

**Progress drafts** は、長時間のエージェントターンを「一時ステータス返信の山」にせず、**1 つの作業中メッセージ**を更新し続けてチャット上で動いて見せ、完了時に最終回答へ変換する仕組み。[[concepts/streaming]] のプレビュー `progress` モードの中身。

## なぜ重要か

ツール多用の長いターンでは、回答テキストをトークン単位で見せるより「**いま何をしているか**」（`🛠️ Bash: run tests`、`🔎 Web Search: ...`）を見せる方が UX が良い。Progress drafts は**単一の下書き**にラベル＋進捗行をまとめ、無関係なステータスメッセージでチャットを汚さない。最終回答は可能なら下書きをその場で置き換え、無理なら安全に新規送信へフォールバックする（テキストを失わない設計）。

## 押さえる点

- 有効化：`channels.<ch>.streaming.mode: "progress"`（通常これだけ）。ラベルは `auto`（カニ/海モチーフのプール）/固定/`false`。
- 進捗行は実行イベントから生成、`toolProgressDetail`（`explain`/`raw`）で詳細度、`maxLines` で行数制限、Slack は `render: "rich"`。
- チャネル別に最適なトランスポート（送信→編集／ネイティブストリーム）。安全な編集が無いチャネルは入力インジケーターか最終のみ。

モード選択・ラベル設定・トラブルシュートは [[sources/concepts/progress-drafts]]。

## 関連

- [[concepts/streaming]] / [[concepts/messages]]
