---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/slack
source_path: raw/docs/channels/slack.md
doc_section: channels
title: "Slack"
ingested: 2026-06-14
tags: [slack, channel, bolt, socket-mode, mpim, workspace-app]
related:
  - "[[channels/slack]]"
  - "[[concepts/channel-routing]]"
  - "[[concepts/pairing]]"
---

# Slack（解説）

> 原典: `raw/docs/channels/slack.md` ・ https://docs.openclaw.ai/ja-JP/channels/slack

## 一言まとめ

Slack アプリ連携で **DM とチャンネル**に本番対応する同梱チャネル（Bolt SDK）。既定は **Socket Mode**（WebSocket）、HTTP Request URLs も可。DM 既定はペアリング。

## 位置づけ

[[channels/slack]] の詳細ソース。[[concepts/channel-routing]] の主要チャネルで、複数人 DM（MPIM）はグループ扱い（[[concepts/messages]] のグループポリシー）。

## 仕組み・ふるまい

- **トランスポート**：Bolt SDK・Socket Mode（既定、外向き WebSocket でポート開放不要）or HTTP Request URLs。ワークスペースアプリ。
- **ネイティブコマンド**：Slack は組み込みコマンドごとに 1 つの Slack スラッシュコマンド作成が必要（自動削除されない）。⚠️ Slack は `/status` 予約のため `/agentstatus` を登録。引数メニューは一時 Block Kit ボタン。

## 設定・使い方の要点

- `channels.slack.*`：Bot/App トークン・`dmPolicy`・`slashCommand`（`/openclaw` 形式）・`native`・複数アカウント。Slack アプリを作成し scopes/イベントを設定。

## 注意点・落とし穴

- 複数人 DM（MPIM）はグループとしてルーティング（グループメンション動作・グループセッション）。dock コマンド `/dock-slack`。

## 用語と略称

- **Bolt SDK** = Slack 公式のアプリフレームワーク
- **Socket Mode** = 外向き WebSocket で受信する方式（公開 URL 不要）
- **MPIM** = Multi-Person IM（複数人 DM）
- **Block Kit** = Slack の UI コンポーネント

## 関連ページ

- [[channels/slack]] — 対応するチャネルページ
- [[concepts/channel-routing]] / [[concepts/pairing]] / [[concepts/slash-commands]] / [[concepts/messages]]
