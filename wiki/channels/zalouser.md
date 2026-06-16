---
type: channel
aliases: [Zalo Personal, Zalo 個人用, zalouser]
tags: [messaging, channel, zalo, unofficial, plugin]
related:
  - "[[concepts/channel-routing]]"
  - "[[components/plugin-system]]"
  - "[[components/gateway]]"
  - "[[concepts/messages]]"
sources:
  - "[[sources/plugins/zalouser]]"
updated: 2026-06-15
---

# Zalo Personal（zalouser）

**Zalo Personal** は、ベトナムの主要メッセージングアプリ **Zalo の個人ユーザーアカウント**を OpenClaw に繋ぐチャネル。Plugin（`@openclaw/zalouser`、ネイティブ `zca-js`）として提供され、QR ログインでアカウントを連携してメッセージを送受信する。チャネル id は `zalouser`（`zalo` は将来の公式 API 用に予約）。

> ℹ️ チャネルは実体としては [[components/plugin-system]] の `registerChannel` で提供され、[[concepts/channel-routing]] の下で他チャネルと同じ [[concepts/messages]] パイプライン・[[concepts/session]]・[[concepts/pairing]] を通る。

## 設定・使い方

- インストール：`openclaw plugins install @openclaw/zalouser` → Gateway 再起動（リモートなら Gateway ホストで）。
- 設定は **`channels.zalouser`**（`plugins.entries.*` ではない）：`enabled`・`dmPolicy: "pairing"` 等。
- CLI：`openclaw channels login --channel zalouser`（QR）／`logout`／`status --probe`／`message send --channel zalouser --target <threadId> --message "..."`。
- エージェントツール `zalouser`：`send`/`image`/`link`/`friends`/`groups`/`me`/`status`/`react`。

## 注意点

⚠️ **非公式自動化**（`zca-js`）であり、Zalo 側のポリシーによりアカウント停止/BAN の可能性がある（自己責任）。これは公式 API 統合の `zalo`（別チャネル、将来）とは区別される。詳細は [[sources/plugins/zalouser]]。

## 関連

- [[concepts/channel-routing]] / [[components/plugin-system]] / [[components/gateway]] / [[concepts/messages]]
- [[concepts/pairing]] / [[sources/plugins/zalouser]]
