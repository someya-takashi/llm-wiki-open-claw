---
type: channel
aliases: [Slack]
tags: [messaging, channel, slack, socket-mode, mpim]
related:
  - "[[components/gateway]]"
  - "[[concepts/channel-routing]]"
sources:
  - "[[sources/channels/slack]]"
updated: 2026-06-14
---

# Slack

**Slack** は、Bolt SDK を使った Slack アプリ連携で **DM とチャンネル**に本番対応する同梱チャネル。既定は **Socket Mode**（外向き WebSocket、公開 URL 不要）。[[concepts/channel-routing]] に乗り、[[components/gateway]] が所有する。

## 要点

- **トランスポート**：Bolt SDK・Socket Mode（既定）or HTTP Request URLs。ワークスペースアプリ。
- **DM ポリシー**：既定**ペアリング**（[[concepts/pairing]]）。
- **複数人 DM（MPIM）はグループ扱い**——グループメンション動作・グループセッション（[[concepts/messages]]）。
- **ネイティブコマンド**：組み込みコマンドごとに Slack スラッシュコマンドを作成。`/status` は予約のため `/agentstatus`。設定 `channels.slack.*`。

詳細・セットアップは [[sources/channels/slack]]。

## 関連

- [[concepts/channel-routing]] / [[concepts/pairing]] / [[concepts/messages]] / [[components/gateway]]
- [[channels/discord]] / [[channels/telegram]] / [[channels/whatsapp]]
