---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/images
source_path: raw/docs/nodes/images.md
doc_section: nodes
title: "画像とメディアのサポート"
ingested: 2026-06-14
tags: [media, image, send, whatsapp, inbound-media, template-vars]
related:
  - "[[concepts/media-understanding]]"
  - "[[concepts/messages]]"
  - "[[channels/whatsapp]]"
---

# 画像とメディアのサポート（解説）

> 原典: `raw/docs/nodes/images.md` ・ https://docs.openclaw.ai/ja-JP/nodes/images

## 一言まとめ

メディアの**送受信ルール**（主に WhatsApp Web 経由）。`openclaw message send --media` での送信、自動返信へのメディア添付、受信メディアのテンプレート変数（`{{MediaPath}}`/`{{MediaUrl}}`）化を扱う。

## 位置づけ

[[concepts/messages]] の送受信パイプラインのメディア面で、受信側は [[concepts/media-understanding]] が要約する前段。チャネル固有挙動は [[channels/whatsapp]]。

## 仕組み・ふるまい

- **送信**：`openclaw message send --media <path-or-url> [--message <caption>]`（`--dry-run`/`--json`）。画像は `channels.whatsapp.mediaMaxMb`（既定 50MB）目標に JPEG 再圧縮（最大辺 2048px）、音声/動画は 16MB パススルー（音声は `ptt: true` のボイスメモ）、ドキュメントは 100MB。MP4＋`gifPlayback: true` で GIF 風ループ。
- **受信（コマンド）**：受信メディアを一時ファイルにダウンロードし `{{MediaUrl}}`（擬似 URL）/`{{MediaPath}}`（ローカル一時パス）を公開。Docker サンドボックス有効時は `media/inbound/<filename>` の相対パスに書き換え。メディア理解（`tools.media.*`）はテンプレート処理前に走り `[Image]`/`[Audio]`/`[Video]` を `Body` に挿入。

## 設定・使い方の要点

- 送信上限：画像 `mediaMaxMb`（既定 50MB）、音声/動画 16MB、ドキュメント 100MB。MIME 検出はマジックバイト→ヘッダー→拡張子。
- 理解上限：画像 10MB・音声 20MB・動画 50MB（超過は理解スキップ、返信は元本文で続行）。

## 注意点・落とし穴

- サイズ超過/読み取り不能なメディアは明確なエラーをログし返信スキップ。既定では最初に一致した 1 添付のみ理解（複数は `tools.media.<cap>.attachments`）。

## 用語と略称

- **Baileys** = WhatsApp Web の非公式実装ライブラリ
- **ptt** = Push-To-Talk（ボイスメモフラグ）
- **`{{MediaPath}}` / `{{MediaUrl}}`** = 受信メディアのローカルパス/擬似 URL テンプレート変数
- **MIME** = メディアタイプ識別子

## 関連ページ

- [[concepts/media-understanding]] / [[sources/nodes/media-understanding]] / [[sources/nodes/audio]]
- [[concepts/messages]] / [[sources/nodes/camera]] / [[channels/whatsapp]]
