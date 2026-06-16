---
type: channel
aliases: [Telegram]
tags: [messaging, channel, telegram, grammy, groups]
related:
  - "[[components/gateway]]"
  - "[[concepts/channel-routing]]"
sources:
  - "[[sources/channels/telegram]]"
updated: 2026-06-14
---

# Telegram

**Telegram** は、grammY 経由の Bot API で **DM とグループ**に本番対応する同梱チャネル。既定はロングポーリング（公開 URL 不要）で、**最速セットアップ**（@BotFather のボットトークンのみ）。[[concepts/channel-routing]] に乗り、[[components/gateway]] が所有する。

## 要点

- **トランスポート**：grammY（Bot API）。既定ロングポーリング、任意 webhook。
- **DM ポリシー**：既定**ペアリング**（[[concepts/pairing]]）。
- **グループ/トピック**：`channels.telegram.groups.<chatId>`（メンション要求・トピック単位の上書き）。`/focus` でトピックをバインド。
- Markdown 画像はメディア返信に変換、ボイスメモのメンション preflight（[[sources/nodes/audio]]）。

詳細・セットアップは [[sources/channels/telegram]]。

## 関連

- [[concepts/channel-routing]] / [[concepts/pairing]] / [[concepts/channel-docking]] / [[components/gateway]]
- [[channels/discord]] / [[channels/slack]] / [[channels/whatsapp]]
