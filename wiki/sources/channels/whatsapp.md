---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/whatsapp
source_path: raw/docs/channels/whatsapp.md
doc_section: channels
title: "WhatsApp"
ingested: 2026-06-14
tags: [whatsapp, channel, baileys, qr-pairing, on-demand, media]
related:
  - "[[channels/whatsapp]]"
  - "[[concepts/channel-routing]]"
  - "[[concepts/architecture]]"
---

# WhatsApp（解説）

> 原典: `raw/docs/channels/whatsapp.md` ・ https://docs.openclaw.ai/ja-JP/channels/whatsapp

## 一言まとめ

WhatsApp Web（Baileys）経由で本番対応する**最も人気**のチャネル。Gateway がリンク済みセッションを所有し、QR ペアリングで連携する。Plugin はオンデマンドインストール。

## 位置づけ

[[channels/whatsapp]] の詳細ソース。[[concepts/channel-routing]] の主要チャネルで、**単一 Baileys セッション**は [[concepts/architecture]] の「ホストごと 1 Gateway」不変条件の代表例。メディア送受信は [[sources/nodes/images]]。

## 仕組み・ふるまい

- **トランスポート**：Baileys（WhatsApp Web の非公式実装）。Gateway がリンク済みセッションを所有（状態をディスク保存）。
- **オンデマンドインストール**：`openclaw onboard`/`openclaw channels add --channel whatsapp` が初回に Plugin インストールを促す。Gateway はチャネルが実際にアクティブなときだけ WhatsApp ランタイムを読み込む。
- メディア：画像は JPEG 再圧縮（`mediaMaxMb` 既定 50MB）、音声はボイスメモ（`ptt`）、GIF 風 MP4（[[sources/nodes/images]]）。

## 設定・使い方の要点

- `channels.whatsapp.*`：`enabled`・`dmPolicy`・`allowFrom`・`mediaMaxMb`。セットアップは QR ペアリング（`openclaw channels login --channel whatsapp` 等）。

## 注意点・落とし穴

- ⚠️ 非公式（Baileys）のため状態管理が重く QR 再ペアリングが要ることがある。単一セッションなので 1 Gateway = 1 WhatsApp アカウント（[[concepts/architecture]]）。

## 用語と略称

- **Baileys** = WhatsApp Web の非公式 TypeScript 実装
- **QR ペアリング** = QR コードでアカウントをリンクする連携
- **リンク済みセッション** = WhatsApp Web の連携状態
- **ptt** = Push-To-Talk（ボイスメモ）

## 関連ページ

- [[channels/whatsapp]] — 対応するチャネルページ
- [[concepts/channel-routing]] / [[concepts/architecture]] / [[concepts/pairing]] / [[sources/nodes/images]]
