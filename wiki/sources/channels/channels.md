---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels
source_path: raw/docs/channels/channels.md
doc_section: channels
title: "チャットチャネル（一覧）"
ingested: 2026-06-14
tags: [channels, messaging, catalog, dm-policy, groups, delivery]
related:
  - "[[concepts/channel-routing]]"
  - "[[components/gateway]]"
  - "[[components/plugin-system]]"
---

# チャットチャネル（一覧）（解説）

> 原典: `raw/docs/channels/channels.md` ・ https://docs.openclaw.ai/ja-JP/channels
>
> ℹ️ `channels/` セクションのランディング。25 以上のチャネルへのカタログ。

## 一言まとめ

OpenClaw が対応する**全チャットチャネルのカタログ**。各チャネルは Gateway 経由で接続し、テキストはどこでも、メディア/リアクションはチャネルごとに異なる。

## 位置づけ

[[concepts/channel-routing]] の中核ソース。各チャネルは [[components/gateway]] が所有し、多くは [[components/plugin-system]] の Plugin。

## 仕組み・ふるまい（主要チャネル）

- **同梱・主要**：[[channels/discord]]（Bot API）・[[channels/slack]]（Bolt/Socket Mode）・[[channels/telegram]]（grammY）・[[channels/whatsapp]]（Baileys/QR、オンデマンド）。
- **同梱 Plugin**：Feishu/QQ/Signal/Synology/Nostr/Tlon/Twitch/Zalo・[[channels/zalouser]]/MS Teams。
- **ダウンロード Plugin**：Google Chat/LINE/Matrix/Mattermost/Nextcloud Talk。**外部**：WeChat/Yuanbao。
- **その他**：iMessage（`imsg` ブリッジ）・IRC・[[components/webchat]]・Voice Call（[[sources/plugins/voice-call]]）。

## 設定・使い方の要点

- チャネルは同時実行可（チャットごとにルーティング）。最速は **Telegram**（ボットトークンのみ）、WhatsApp は QR ペアリング＋状態をディスク保存。
- DM ペアリングと許可リストは安全のため強制（[[concepts/pairing]]・[[concepts/security]]）。グループ挙動はチャネル依存。

## 注意点・落とし穴

- Slack の複数人 DM（MPIM）はグループとして扱う。Telegram の Markdown 画像構文は可能ならメディア返信に変換。モデルプロバイダーは別物（[[concepts/model-providers]]）。

## 用語と略称

- **チャネル（channel）** = メッセージングアプリ統合
- **DM ポリシー** = ダイレクトメッセージの受け入れ方針（既定 pairing）
- **MPIM** = Multi-Person IM（Slack の複数人 DM）
- **Baileys / grammY / Bolt** = WhatsApp / Telegram / Slack のライブラリ

## 関連ページ

- [[concepts/channel-routing]] — 対応する概念ページ
- [[channels/discord]] / [[channels/slack]] / [[channels/telegram]] / [[channels/whatsapp]]
- [[components/gateway]] / [[components/plugin-system]]
