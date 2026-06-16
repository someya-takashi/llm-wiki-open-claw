---
type: concept
aliases: [Streaming, ストリーミング, chunking, チャンク化]
tags: [streaming, chunking, block-streaming, preview]
related:
  - "[[concepts/messages]]"
  - "[[concepts/progress-drafts]]"
  - "[[concepts/retry]]"
sources:
  - "[[sources/concepts/streaming]]"
updated: 2026-06-14
---

# ストリーミングとチャンク化

OpenClaw のストリーミングは **2 つの独立した層**でできている：①**ブロックストリーミング**（完成した「ブロック」を通常のチャネルメッセージとして送出）と②**プレビューストリーミング**（生成中の一時メッセージを編集更新）。**真のトークン差分ストリーミングは無く**、プレビューはメッセージベース（送信＋編集/追記）。

## なぜ重要か

チャネルごとに文字数上限・編集可否・ネイティブストリーミング API が違うため、「進行に応じて見せる」体験を統一的に扱う層が要る。`EmbeddedBlockChunker`（min/max 境界・区切り優先・コードフェンス保護）が[[concepts/messages]] の送信段を支え、ツール多用ターンの「いま何をしているか」表示は [[concepts/progress-drafts]]（`progress` モード）が担う。**ブロックストリーミングとプレビューは二重送信を避けるため排他的に働く**のが運用上の肝。

## 押さえる点

- ブロック：`agents.defaults.blockStreaming*`（既定 off、Telegram 以外は `*.blockStreaming: true` 必須）。`blockStreamingBreak` は `text_end`（逐次）/`message_end`（まとめて）。`coalesce` で 1 行スパム抑制、`humanDelay` で人間らしい間隔。
- プレビュー：`channels.<ch>.streaming`（`off`/`partial`/`block`/`progress`）。`streaming.mode: "block"` は**プレビュー**であって通常ブロック配信ではない。
- レガシーキーは `openclaw doctor --fix` で `streaming.*` へ移行。

全制御キー・チャネル別対応表・ツール進捗は [[sources/concepts/streaming]]。

## 関連

- [[concepts/messages]] / [[concepts/progress-drafts]] / [[concepts/retry]]
