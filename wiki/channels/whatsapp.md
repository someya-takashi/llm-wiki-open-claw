---
type: channel
aliases: [WhatsApp]
tags: [messaging, channel, whatsapp, baileys, qr-pairing]
related:
  - "[[components/gateway]]"
  - "[[concepts/channel-routing]]"
sources:
  - "[[sources/channels/whatsapp]]"
updated: 2026-06-14
---

# WhatsApp

**WhatsApp** は、WhatsApp Web（Baileys）経由で本番対応する**最も人気**のチャネル。Gateway がリンク済みセッションを所有し、**QR ペアリング**で連携する。[[concepts/channel-routing]] に乗り、[[components/gateway]] が所有する。

## 要点

- **トランスポート**：Baileys（WhatsApp Web 非公式実装）。Gateway がリンク済みセッションを所有（状態をディスク保存）。
- **オンデマンドインストール**：初回に Plugin インストールを促し、チャネルがアクティブなときだけランタイムを読み込む。
- **単一セッション**：1 Gateway = 1 WhatsApp アカウント——[[concepts/architecture]] の「ホストごと 1 Gateway」不変条件の代表例。
- メディア：画像は JPEG 再圧縮（`mediaMaxMb` 既定 50MB）、音声はボイスメモ（[[sources/nodes/images]]）。設定 `channels.whatsapp.*`。

詳細・セットアップは [[sources/channels/whatsapp]]。

## 関連

- [[concepts/channel-routing]] / [[concepts/architecture]] / [[concepts/pairing]] / [[components/gateway]]
- [[channels/discord]] / [[channels/slack]] / [[channels/telegram]]
