---
type: concept
aliases: [Cron, スケジュールされたタスク, Scheduled Tasks, cron jobs]
tags: [cron, scheduled-tasks, automation, webhook, isolated-session]
related:
  - "[[concepts/automation]]"
  - "[[concepts/heartbeat]]"
  - "[[concepts/tasks]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/automation/cron-jobs]]"
updated: 2026-06-14
---

# Cron（スケジュールされたタスク）

Cron（cron, 時刻・間隔・cron 式でエージェントターンを予約する仕組み）は、[[concepts/automation]] の「**正確なタイミング**」担当。単発リマインダー（`--at 20m`）から定期レポート（cron 式）まで、必要なら別モデル・分離セッションで実行する。

## なぜ重要か

「毎日 9:00 にレポート」「20 分後にリマインド」のような**時刻が本質の作業**は、おおよその [[concepts/heartbeat]] では不適切。Cron は正確なタイミング・クリーンな分離セッション・常にタスク記録（[[concepts/tasks]]）・柔軟な配信を提供し、エージェントを「決まった時間に決まった仕事をする」存在にする。

## 仕組み（要点）

- **タイプ**：`at`（単発）/`every`（固定間隔）/`cron`（5/6 フィールド式＋`--tz`）。day-of-month と day-of-week は OR。
- **実行スタイル（`--session`）**：`main`（次の Heartbeat ターン）/`isolated`（`cron:<jobId>` 専用）/`current`/`session:<id>`。
- **配信**：`announce`（フォールバック配信）/`webhook`（完了を URL に POST）/`none`。
- CLI `openclaw cron add|list|show|run|enable|disable|remove`、設定 `cron.*`。Gmail PubSub で受信メールトリガーも。詳細は [[sources/automation/cron-jobs]]。

## 既存 wiki とのつながり

Cron と [[concepts/heartbeat]] は自動化の二本柱——前者は「正確な時刻・分離文脈」、後者は「おおよそ・メイン文脈」。`--session main` の cron は実際には Heartbeat ターンに乗る。各実行は [[concepts/tasks]] の台帳に記録され、[[components/gateway]] が永続化・実行する。常に効く指示は [[sources/automation/standing-orders]] と組み合わせて週次サイクル等を回す。

## 代表ソース

- [[sources/automation/cron-jobs]] — スケジュールタイプ・実行スタイル・配信・Gmail 連携

## 関連ページ

- [[concepts/automation]] / [[concepts/heartbeat]] / [[concepts/tasks]]
- [[sources/automation/taskflow]] / [[components/gateway]]
