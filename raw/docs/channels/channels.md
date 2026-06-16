---
title: "チャットチャネル"
source: "https://docs.openclaw.ai/ja-JP/channels"
author:
published:
created: 2026-06-14
description: "OpenClawが接続できるメッセージングプラットフォーム"
tags:
  - "clippings"
---
OpenClaw は、すでに使用している任意のチャットアプリであなたと会話できます。各チャネルは Gateway 経由で接続します。 テキストはすべての場所でサポートされています。メディアとリアクションはチャネルによって異なります。

## 配信に関する注記

- `![alt](url)` など、Markdown 画像構文を含む Telegram 返信は、 可能な場合、最終的な送信経路でメディア返信に変換されます。
- Slack の複数人 DM はグループチャットとしてルーティングされるため、グループポリシー、メンション 動作、グループセッションルールが MPIM 会話に適用されます。
- WhatsApp のセットアップはオンデマンドインストールです。オンボーディングでは、 Plugin パッケージがインストールされる前にセットアップフローを表示でき、Gateway はチャネルが実際にアクティブな場合にのみ WhatsApp ランタイムを読み込みます。

## サポートされているチャネル

- [Discord](https://docs.openclaw.ai/ja-JP/channels/discord) - Discord Bot API + Gateway。サーバー、チャネル、DM をサポートします。
- [Feishu](https://docs.openclaw.ai/ja-JP/channels/feishu) - WebSocket 経由の Feishu/Lark ボット（同梱 Plugin）。
- [Google Chat](https://docs.openclaw.ai/ja-JP/channels/googlechat) - HTTP Webhook 経由の Google Chat API アプリ（ダウンロード可能な Plugin）。
- [iMessage](https://docs.openclaw.ai/ja-JP/channels/imessage) - サインイン済み Mac 上の `imsg` ブリッジ（または Gateway が別の場所で実行される場合は SSH ラッパー）経由のネイティブ macOS 統合。返信、タップバック、エフェクト、添付ファイル、グループ管理のためのプライベート API アクションを含みます。ホスト権限と Messages へのアクセスが適合する場合、新しい OpenClaw iMessage セットアップで推奨されます。
- [IRC](https://docs.openclaw.ai/ja-JP/channels/irc) - クラシックな IRC サーバー。ペアリング/許可リスト制御付きのチャネル + DM。
- [LINE](https://docs.openclaw.ai/ja-JP/channels/line) - LINE Messaging API ボット（ダウンロード可能な Plugin）。
- [Matrix](https://docs.openclaw.ai/ja-JP/channels/matrix) - Matrix プロトコル（ダウンロード可能な Plugin）。
- [Mattermost](https://docs.openclaw.ai/ja-JP/channels/mattermost) - Bot API + WebSocket。チャネル、グループ、DM（ダウンロード可能な Plugin）。
- [Microsoft Teams](https://docs.openclaw.ai/ja-JP/channels/msteams) - Bot Framework。エンタープライズサポート（同梱 Plugin）。
- [Nextcloud Talk](https://docs.openclaw.ai/ja-JP/channels/nextcloud-talk) - Nextcloud Talk 経由のセルフホスト型チャット（同梱 Plugin）。
- [Nostr](https://docs.openclaw.ai/ja-JP/channels/nostr) - NIP-04 経由の分散型 DM（同梱 Plugin）。
- [QQ Bot](https://docs.openclaw.ai/ja-JP/channels/qqbot) - QQ Bot API。プライベートチャット、グループチャット、リッチメディア（同梱 Plugin）。
- [Signal](https://docs.openclaw.ai/ja-JP/channels/signal) - signal-cli。プライバシー重視。
- [Slack](https://docs.openclaw.ai/ja-JP/channels/slack) - Bolt SDK。ワークスペースアプリ。
- [Synology Chat](https://docs.openclaw.ai/ja-JP/channels/synology-chat) - outgoing+incoming Webhook 経由の Synology NAS Chat（同梱 Plugin）。
- [Telegram](https://docs.openclaw.ai/ja-JP/channels/telegram) - grammY 経由の Bot API。グループをサポートします。
- [Tlon](https://docs.openclaw.ai/ja-JP/channels/tlon) - Urbit ベースのメッセンジャー（同梱 Plugin）。
- [Twitch](https://docs.openclaw.ai/ja-JP/channels/twitch) - IRC 接続経由の Twitch チャット（同梱 Plugin）。
- [Voice Call](https://docs.openclaw.ai/ja-JP/plugins/voice-call) - Plivo または Twilio 経由の電話機能（Plugin、別途インストール）。
- [WebChat](https://docs.openclaw.ai/ja-JP/web/webchat) - WebSocket 上の Gateway WebChat UI。
- [WeChat](https://docs.openclaw.ai/ja-JP/channels/wechat) - QR ログイン経由の Tencent iLink Bot Plugin。プライベートチャットのみ（外部 Plugin）。
- [WhatsApp](https://docs.openclaw.ai/ja-JP/channels/whatsapp) - 最も人気があります。Baileys を使用し、QR ペアリングが必要です。
- [Yuanbao](https://docs.openclaw.ai/ja-JP/channels/yuanbao) - Tencent Yuanbao ボット（外部 Plugin）。
- [Zalo](https://docs.openclaw.ai/ja-JP/channels/zalo) - Zalo Bot API。ベトナムで人気のメッセンジャー（同梱 Plugin）。
- [Zalo Personal](https://docs.openclaw.ai/ja-JP/channels/zalouser) - QR ログイン経由の Zalo 個人アカウント（同梱 Plugin）。

## 注記

- チャネルは同時に実行できます。複数を設定すると、OpenClaw がチャットごとにルーティングします。
- 最速のセットアップは通常 **Telegram** です（シンプルなボットトークン）。WhatsApp は QR ペアリングが必要で、 より多くの状態をディスクに保存します。
- グループの動作はチャネルによって異なります。 [グループ](https://docs.openclaw.ai/ja-JP/channels/groups) を参照してください。
- DM ペアリングと許可リストは安全のために強制されます。 [セキュリティ](https://docs.openclaw.ai/ja-JP/gateway/security) を参照してください。
- トラブルシューティング: [チャネルのトラブルシューティング](https://docs.openclaw.ai/ja-JP/channels/troubleshooting) 。
- モデルプロバイダーは別途文書化されています。 [モデルプロバイダー](https://docs.openclaw.ai/ja-JP/providers/models) を参照してください。