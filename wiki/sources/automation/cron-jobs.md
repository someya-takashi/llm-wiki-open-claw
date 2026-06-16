---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/automation/cron-jobs
source_path: raw/docs/automation/cron-jobs.md
doc_section: automation
title: "スケジュールされたタスク（Cron）"
ingested: 2026-06-14
tags: [cron, scheduled-tasks, automation, webhook, isolated-session, gmail]
related:
  - "[[concepts/cron]]"
  - "[[concepts/automation]]"
  - "[[concepts/heartbeat]]"
---

# スケジュールされたタスク（Cron）（解説）

> 原典: `raw/docs/automation/cron-jobs.md` ・ https://docs.openclaw.ai/ja-JP/automation/cron-jobs

## 一言まとめ

時刻・間隔・cron 式で**エージェントターンを正確なタイミングで予約**する仕組み。単発リマインダーから定期レポートまで、分離セッションや別モデルでも実行でき、結果をチャネル/webhook/無音で配信する。

## 位置づけ

[[concepts/cron]] の中核ソース。[[concepts/automation]] の「正確なタイミング」担当で、おおよその定期処理 [[concepts/heartbeat]] と対になる。各実行は [[concepts/tasks]] のタスク記録を作る。

## 仕組み・ふるまい

- **スケジュールタイプ**：`at`（`--at`、ISO 8601 or `20m` 相対の単発）/`every`（`--every`、固定間隔）/`cron`（`--cron`、5/6 フィールド＋`--tz`）。day-of-month と day-of-week は **OR ロジック**。
- **実行スタイル（`--session`）**：`main`（次の Heartbeat ターン）/`isolated`（専用 `cron:<jobId>`、レポート向け）/`current`（作成時バインド）/`session:<id>`（永続名前付き）。
- **配信モード**：`announce`（エージェントが送らなければ最終テキストをフォールバック配信）/`webhook`（完了イベントを URL に POST）/`none`。

## 設定・使い方の要点

- CLI：`openclaw cron add`（`--at`/`--every`/`--cron`＋`--tz`/`--session`/`--model`/`--thinking`/配信）、`list`/`show`/`run`/`enable`/`disable`/`remove`。設定は `cron.*`。
- Gmail PubSub 連携（受信メールトリガー）はウィザード設定推奨。分離ジョブのペイロードでモデル/thinking 上書き可。

## 注意点・落とし穴

- Webhook 配信は認証（`cron.webhookToken`）を設定可。トラブル時は `openclaw cron run <id>` で手動実行して確認。メインセッションジョブは Heartbeat に乗るので即時ではない。

## 用語と略称

- **cron 式** = `分 時 日 月 曜日` 形式のスケジュール記法
- **`--at` / `--every`** = 単発タイムスタンプ / 固定間隔
- **分離セッション（isolated）** = `cron:<jobId>` の専用セッション
- **PubSub** = Google のイベント通知（Gmail 受信トリガー）

## 関連ページ

- [[concepts/cron]] — 対応する概念ページ
- [[concepts/automation]] / [[concepts/heartbeat]] / [[concepts/tasks]]
- [[sources/automation/taskflow]] / [[sources/automation/standing-orders]]
