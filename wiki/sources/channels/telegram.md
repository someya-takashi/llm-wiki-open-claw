---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/telegram
source_path: raw/docs/channels/telegram.md
doc_section: channels
title: "Telegram"
ingested: 2026-06-14
tags: [telegram, channel, grammy, bot-api, long-polling, groups, topics]
related:
  - "[[channels/telegram]]"
  - "[[concepts/channel-routing]]"
  - "[[concepts/pairing]]"
---

# Telegram（解説）

> 原典: `raw/docs/channels/telegram.md` ・ https://docs.openclaw.ai/ja-JP/channels/telegram

## 一言まとめ

grammY 経由の Bot API で **DM とグループ**に本番対応する同梱チャネル。既定はロングポーリング、webhook も可。**最速セットアップ**（ボットトークンのみ）。

## 位置づけ

[[channels/telegram]] の詳細ソース。[[concepts/channel-routing]] の主要チャネルで、DM 既定はペアリング（[[concepts/pairing]]）。グループ/トピックの細かい制御を持つ。

## 仕組み・ふるまい

- **トランスポート**：grammY（Bot API）。既定ロングポーリング（公開 URL 不要）、任意で webhook。
- **グループ/トピック**：`channels.telegram.groups.<chatId>`（メンション要求・音声 preflight 無効化 `disableAudioPreflight` 等）、トピック単位の上書き。`/focus` でトピック/会話をバインド。
- Markdown 画像構文 `![](url)` は可能なら最終送信でメディア返信に変換。

## 設定・使い方の要点

- `channels.telegram.*`：ボットトークン（@BotFather）・`dmPolicy`・`allowFrom`・`native`/`nativeSkills`・`groups`・複数アカウント。

## 注意点・落とし穴

- グループでメンション必須にするとボイスメモは preflight 文字起こしでメンション判定（[[sources/nodes/audio]]）。dock コマンド `/dock-telegram`。

## 用語と略称

- **grammY** = Telegram Bot API の TypeScript フレームワーク
- **ロングポーリング** = サーバーに繰り返し問い合わせて受信する方式
- **webhook** = サーバーが push 受信する方式
- **トピック（topic）** = Telegram グループ内のスレッド

## 関連ページ

- [[channels/telegram]] — 対応するチャネルページ
- [[concepts/channel-routing]] / [[concepts/pairing]] / [[concepts/channel-docking]] / [[sources/nodes/audio]]
