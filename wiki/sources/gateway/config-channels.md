---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/config-channels
source_path: raw/docs/gateway/config-channels.md
doc_section: gateway
title: "設定 — チャンネル"
ingested: 2026-06-14
tags: [configuration, channels, dm-policy, mention-gating, multi-account]
related:
  - "[[concepts/configuration]]"
  - "[[concepts/channel-routing]]"
  - "[[components/gateway]]"
---

# 設定 — チャンネル（解説）

> 原典: `raw/docs/gateway/config-channels.md` ・ https://docs.openclaw.ai/ja-JP/gateway/config-channels

## 一言まとめ

`channels.*` 配下の**チャネルごとの設定キー**を扱うリファレンス。DM/グループのアクセス制御、複数アカウント、メンションゲート、各同梱チャネル（WhatsApp/Telegram/Discord/Slack/Signal/iMessage/Matrix/Teams/IRC…）のキー。

## 位置づけ

[[concepts/configuration]] のチャネル・ドメイン。各チャネルは [[components/gateway]] が所有し、受信メッセージのルーティングは [[concepts/channel-routing]]・[[concepts/multi-agent]] のバインディングへつながる。個別チャネルの専用ページは [[channels/discord]]・[[channels/slack]]・[[channels/telegram]]・[[channels/whatsapp]] 等（[[concepts/channel-routing]] から辿れる）、本ページは横断的なチャネル設定（DM/グループ・メンションゲート・複数アカウント）の入口。

## 仕組み・ふるまい（主なキー群）

- **自動起動**：`channels.<provider>` セクションが存在すれば自動的に起動（`enabled: false` を除く）。
- **DM とグループアクセス**：`dmPolicy`（`pairing`＝既定/不明送信者にワンタイムコード・`allowlist`＝`allowFrom` のみ・`open`＝全許可（`allowFrom: ["*"]` 必須）・`disabled`）、グループは `groupPolicy`＋`groupAllowFrom`。
- **チャネルモデルの上書き**：チャネル単位で使うモデルを変える。
- **チャネルデフォルトと Heartbeat**・**複数アカウント**（`channels.<provider>.accounts.<id>`、`accountId` でエージェントへ振り分け → [[concepts/multi-agent]]）。
- **個別チャネル**：WhatsApp・Telegram・Discord・Google Chat・Slack・Mattermost・Signal・iMessage・Matrix・Microsoft Teams・IRC ほか、それぞれのトークン/権限/挙動キー。
- **グループチャットのメンションゲート**：グループは既定でメンション必須。`agents.list[].groupChat.mentionPatterns`（安全な正規表現）＋ネイティブ @ メンション。`messages.visibleReplies`/`messages.groupChat.visibleReplies`（`automatic`＝レガシー自動返信／`message_tool`＝既定でメッセージツール送信を要求）。
- **コマンド**：チャットコマンドの処理設定。

## 設定・使い方の要点

- **マルチユーザー DM は `dmPolicy: "allowlist"`/`"pairing"` ＋ DM 分離**（`session.dmScope` → [[concepts/session]]）でプライバシーを守る。
- **ヘルス監視**：`gateway.channelHealthCheckMinutes`/`channelStaleEventThresholdMinutes`/`channelMaxRestartsPerHour`、チャネル/アカウント単位の `healthMonitor.enabled`（[[sources/gateway/gateway]] の運用面）。
- チャネルは多くがホットリロード対象（再起動不要 → [[concepts/configuration]]）。

## 注意点・落とし穴

- **メンションゲートはグループの暴発防止の要**。`requireMention: true`（`channels.<provider>.groups`）と `mentionPatterns` を併用。
- DM アクセス制御は WhatsApp 等で**アカウントごとにグローバル**（エージェント単位ではない）。
- 個別チャネルの細かいセットアップ手順は各チャネル専用ドキュメント（`channels/<provider>`、本 wiki では未取り込み）。

## 用語と略称

- **dmPolicy / groupPolicy** = DM / グループの受け入れ方針
- **メンションゲート（mention gating）** = グループで @ メンション時のみ反応させる仕組み
- **visibleReplies** = 可視返信を自動で出すか、メッセージツール送信に限るか
- **accounts** = 1 チャネル内の複数アカウント（番号/ボット）

## 関連ページ

- [[concepts/configuration]] — 設定の全体像
- [[concepts/channel-routing]] / [[concepts/multi-agent]] / [[concepts/session]]
- [[components/gateway]] / [[sources/gateway/config-agents]] / [[sources/gateway/configuration-reference]]
