---
type: channel
aliases: [Discord]
tags: [messaging, channel, discord, guild, dm]
related:
  - "[[components/gateway]]"
  - "[[concepts/channel-routing]]"
sources:
  - "[[sources/channels/discord]]"
updated: 2026-06-14
---

# Discord

**Discord** は、公式 Discord Gateway（Bot API）経由で **DM とギルド（サーバー）チャンネル**に対応する同梱チャネル。他チャネルと同じ [[concepts/channel-routing]] に乗り、[[components/gateway]] が接続を所有する。

## 要点

- **トランスポート**：Discord 公式 Bot API。サーバーチャンネル・DM・スレッド。
- **DM ポリシー**：既定**ペアリング**（[[concepts/pairing]]）——未知の DM 送信者は承認が必要。
- **ネイティブコマンド**：スラッシュコマンド（`/model`/`/think` のインタラクティブピッカー）、ボイスチャンネル（`/vc`）、スレッドバインディング（`/focus`）。[[concepts/slash-commands]]。
- 設定は `channels.discord.*`（トークン・`dmPolicy`・`voice`・`threadBindings`・複数アカウント）。

詳細・セットアップは [[sources/channels/discord]]。

## 関連

- [[concepts/channel-routing]] / [[concepts/pairing]] / [[concepts/slash-commands]] / [[components/gateway]]
- [[channels/slack]] / [[channels/telegram]] / [[channels/whatsapp]]
