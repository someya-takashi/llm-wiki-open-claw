---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/health
source_path: raw/docs/gateway/health.md
doc_section: gateway
title: "ヘルスチェック"
ingested: 2026-06-14
tags: [health, status, probe, channel-health, monitoring]
related:
  - "[[concepts/diagnostics]]"
  - "[[components/gateway]]"
  - "[[sources/gateway/gateway]]"
---

# ヘルスチェック（解説）

> 原典: `raw/docs/gateway/health.md` ・ https://docs.openclaw.ai/ja-JP/gateway/health

## 一言まとめ

接続状態を推測せずに**チャネル接続性と Gateway の健全性を確認**するための短いガイド。`openclaw status` 系と `openclaw health`（実行中 Gateway へのライブヘルスプローブ）の使い分け。

## 位置づけ

[[concepts/diagnostics]] の「生存・接続性確認」面。[[sources/gateway/gateway]]（運用手順書）の Liveness/Readiness を具体化し、失敗時は [[sources/gateway/troubleshooting]] に続く。

## 仕組み・ふるまい

- **クイックチェック**：`openclaw status`（ローカル要約）／`status --all`（完全なローカル診断、貼り付け安全）／`status --deep`（実行中 Gateway にライブ `health probe:true`＋アカウント別チャネルプローブ）／`openclaw health`（WS でヘルススナップショット要求。CLI から直接チャネルソケットには繋がない）／`health --verbose`（ライブプローブ強制）／`health --json`。
- WhatsApp/WebChat で `/status` を単独送信すればエージェントを起動せずステータス返信。
- **ヘルススナップショット**：`ok`/`ts`/`durationMs`/チャネル別ステータス/エージェント可用性/セッションストア要約。Gateway 不達やプローブ失敗で非ゼロ終了。
- **注意**：Discord 等でセッション行の有無はソケット生存性ではない（保存会話状態の読み取り）。ライブ接続性はヘルス/チャネルステータスで見る。

## 設定・使い方の要点

- **ヘルスモニター（自動再起動）**：`gateway.channelHealthCheckMinutes`（既定 5、`0` で無効）・`channelStaleEventThresholdMinutes`（既定 30、チェック間隔以上）・`channelMaxRestartsPerHour`（既定 10）。チャネル/アカウント単位は `channels.<provider>[.accounts.<id>].healthMonitor.enabled`（Discord/Google Chat/iMessage/Teams/Signal/Slack/Telegram/WhatsApp が公開）。
- 再リンク：ステータス 409–515 や `loggedOut` なら `openclaw channels logout && openclaw channels login --verbose`。

## 注意点・落とし穴

- `openclaw health` は既定でキャッシュ済みスナップショットを返すことがある（その後 Gateway がバックグラウンド更新）。確実にライブを取るなら `--verbose`。
- 受信が無い→リンク済み端末がオンラインか・送信者が許可されているか（`allowFrom`・グループのメンションルール）を確認。

## 用語と略称

- **liveness / readiness** = 生存確認 / 受付可能確認
- **probe（プローブ）** = チャネル/モデルへの能動的な到達確認
- **stale** = 一定時間イベントが無く「古く」なった接続（自動再起動の対象）
- **ヘルスモニター** = 古いチャネルを検出して再起動する仕組み

## 関連ページ

- [[concepts/diagnostics]] — 対応する概念ページ
- [[sources/gateway/gateway]] / [[sources/gateway/troubleshooting]] / [[sources/gateway/diagnostics]]
- [[components/gateway]]
