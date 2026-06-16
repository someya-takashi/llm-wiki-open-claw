---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/discord
source_path: raw/docs/channels/discord.md
doc_section: channels
title: "Discord"
ingested: 2026-06-14
tags: [discord, channel, bot-api, guild, dm, native-commands, voice]
related:
  - "[[channels/discord]]"
  - "[[concepts/channel-routing]]"
  - "[[concepts/pairing]]"
---

# Discord（解説）

> 原典: `raw/docs/channels/discord.md` ・ https://docs.openclaw.ai/ja-JP/channels/discord

## 一言まとめ

公式 Discord Gateway（Bot API）経由で **DM とギルドチャンネル**に対応する同梱チャネル。DM は既定でペアリングモード、ネイティブスラッシュコマンド・ボイスチャンネル・スレッドバインディングをサポート。

## 位置づけ

[[channels/discord]] の詳細ソース。[[concepts/channel-routing]] の主要チャネルで、DM 承認は [[concepts/pairing]]、ネイティブコマンドは [[concepts/slash-commands]]。

## 仕組み・ふるまい

- **トランスポート**：Discord Bot API（公式 Gateway）。サーバー（ギルド）チャネル・DM。
- **ネイティブコマンド**：`commands.native` 既定オン。スラッシュコマンド登録（ローカライズ説明可）、`/model`/`/think` 等のインタラクティブピッカー（25 オプション制限内）。
- **ボイス**：`/vc join|leave|status`（`channels.discord.voice`）。**スレッドバインディング**：`/focus` でスレッド/会話をセッションにバインド（`channels.discord.threadBindings`）。

## 設定・使い方の要点

- `channels.discord.*`：トークン・`dmPolicy`（既定 pairing）・`allowFrom`・`native`/`nativeSkills`・`voice`・`threadBindings`・複数アカウント。Bot をサーバーに招待し intents を設定。

## 注意点・落とし穴

- ネイティブコマンドの削除は Discord アプリ側に残ることがある（`native: false` でも）。スレッドバインドコマンドは `threadBindings.enabled` が必要。dock コマンド `/dock-discord`（[[concepts/channel-docking]]）。

## 用語と略称

- **ギルド（guild）** = Discord のサーバー
- **Bot API / Gateway** = Discord の公式ボット接続
- **intents** = ボットが受け取れるイベントの権限
- **スレッドバインディング** = スレッドをセッションに束ねる機能

## 関連ページ

- [[channels/discord]] — 対応するチャネルページ
- [[concepts/channel-routing]] / [[concepts/pairing]] / [[concepts/slash-commands]] / [[concepts/channel-docking]]
