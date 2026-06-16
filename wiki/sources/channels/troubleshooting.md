---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/troubleshooting
source_path: raw/docs/channels/troubleshooting.md
doc_section: channels
title: "チャネルのトラブルシューティング"
ingested: 2026-06-14
tags: [channels, troubleshooting, diagnostics, connectivity, per-channel]
related:
  - "[[concepts/channel-routing]]"
  - "[[concepts/diagnostics]]"
---

# チャネルのトラブルシューティング（解説）

> 原典: `raw/docs/channels/troubleshooting.md` ・ https://docs.openclaw.ai/ja-JP/channels/troubleshooting

## 一言まとめ

接続はできるのに動作が誤っているチャネルの診断手順書。共通の段階的確認に加え、WhatsApp/Telegram/Discord/Slack/iMessage/Signal/QQ のチャネル別チェックを示す。

## 位置づけ

[[concepts/channel-routing]] の運用補助で、[[concepts/diagnostics]]（health/doctor/logs）のチャネル版。ノード側は [[sources/nodes/troubleshooting]]。

## 仕組み・ふるまい

- **段階的確認**：`openclaw channels status --probe`・`openclaw logs --follow`・`openclaw doctor` で接続性と設定を確認。
- **チャネル別**：WhatsApp（QR 再ペアリング/セッション）、Telegram（トークン/ポーリング）、Discord（intents/権限）、Slack（scopes/Socket Mode）、iMessage（imsg/権限）、Signal（signal-cli）、QQ Bot ごとの典型問題。

## 設定・使い方の要点

- まず `channels status --probe` で全チャネルの接続状態を見る。返信が来ない時は許可リスト/メンションゲート/`/deliver` を確認。

## 注意点・落とし穴

- 「接続済みだが応答しない」は多くがメンションゲート（[[concepts/groups]]）・許可リスト（[[concepts/pairing]]）・モデル未許可（[[sources/concepts/models]]）が原因。

## 用語と略称

- **`channels status --probe`** = 各チャネルの接続性を実際に確認するコマンド
- **intents** = Discord のイベント受信権限
- **scopes** = Slack アプリの権限スコープ
- **signal-cli** = Signal 連携の CLI

## 関連ページ

- [[concepts/channel-routing]] / [[concepts/diagnostics]] / [[concepts/groups]]
- [[sources/nodes/troubleshooting]] / [[sources/gateway/troubleshooting]]
