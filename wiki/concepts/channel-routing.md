---
type: concept
aliases: [Channel Routing, チャネルルーティング, Channels, チャットチャネル]
tags: [channel-routing, channels, messaging, routing, dm-policy, groups]
related:
  - "[[concepts/messages]]"
  - "[[concepts/pairing]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/groups]]"
components:
  - "[[components/gateway]]"
  - "[[components/plugin-system]]"
sources:
  - "[[sources/channels/channels]]"
  - "[[sources/channels/channel-routing]]"
  - "[[sources/channels/troubleshooting]]"
  - "[[sources/channels/location]]"
updated: 2026-06-14
---

# チャネルルーティング（Channel Routing）

チャネルルーティング（channel routing, どのメッセージングアプリのどの会話を、どのエージェント/セッションに振り分けるか）は、OpenClaw が「**あなたが既に使っているチャットアプリで会話する**」ための仕組み。各チャネルは [[components/gateway]] 経由で接続し、テキストはどこでも、メディア/リアクションはチャネルごとに対応が異なる。

## チャネルの広がり

OpenClaw は **25 以上のチャネル**に対応する。多くは [[components/plugin-system]] の Plugin（同梱 or ダウンロード）として実装される：

| 主要 | 実装 |
|---|---|
| [[channels/discord]] | Discord Bot API（同梱） |
| [[channels/slack]] | Bolt SDK・Socket Mode（同梱） |
| [[channels/telegram]] | grammY Bot API（同梱） |
| [[channels/whatsapp]] | Baileys（WhatsApp Web）・QR ペアリング（オンデマンド） |

ほかに Feishu/Google Chat/iMessage（`imsg` ブリッジ）/IRC/LINE/Matrix/Mattermost/MS Teams/Nextcloud Talk/Nostr/QQ/Signal/Synology/Tlon/Twitch/WeChat/Yuanbao/Zalo（[[channels/zalouser]]）/WebChat（[[components/webchat]]）など。

## なぜ重要か

「専用アプリを入れさせる」のでなく「ユーザーの居場所（WhatsApp/Telegram/Slack…）に出向く」のが OpenClaw のマルチチャネル思想の核。Gateway が全チャネルを所有して**チャットごとにルーティング**するため、複数チャネルを同時運用でき、どこから話しかけても同じエージェント・同じ [[concepts/session]] にアクセスできる。返信先の付け替えは [[concepts/channel-docking]]。

## 共通のふるまい

- **DM ポリシー**：多くのチャネルは DM 既定が **ペアリング**（[[concepts/pairing]]）——未知の送信者は承認が要る。許可リストと併せて [[concepts/security]] の境界を作る。
- **グループ**：グループの挙動はチャネルごとに異なる（メンションゲート・許可リスト・アクセスグループ・ブロードキャスト）——詳細は [[concepts/groups]]。Slack の複数人 DM はグループとして扱う。返信の決定的ルーティング（モデルは宛先を選べない）の仕組みは [[sources/channels/channel-routing]]。
- **メディア配信**：Markdown 画像構文は可能なら最終送信でメディア返信に変換（[[concepts/messages]] のパイプライン）。
- **マルチアカウント/マルチエージェント**：チャネル×アカウント×エージェントのバインディング（[[concepts/multi-agent]]）。

## 既存 wiki とのつながり

受信→返信の処理は [[concepts/messages]] が、配信先の決定はバインディング（[[concepts/multi-agent]]）が担い、チャネルはその「入口/出口」。チャネル Plugin の作り方は [[sources/plugins/sdk-channel-plugins]]。最速セットアップは Telegram（ボットトークンのみ）、WhatsApp は QR ペアリングが要る。

## 代表ソース

- [[sources/channels/channels]] — 全チャネルのカタログと配信注記
- [[sources/channels/channel-routing]] — 決定的ルーティング・セッションキー・宛先プレフィックス
- [[sources/channels/troubleshooting]] — チャネル別の診断 / [[sources/channels/location]] — 位置共有の解析

## 関連ページ

- [[concepts/messages]] / [[concepts/pairing]] / [[concepts/multi-agent]] / [[concepts/groups]] / [[concepts/channel-docking]]
- [[components/gateway]] / [[components/plugin-system]]
- [[channels/discord]] / [[channels/slack]] / [[channels/telegram]] / [[channels/whatsapp]]
