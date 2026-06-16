---
type: concept
aliases: [Tasks, タスク, Background Tasks, Task Flow, タスク台帳]
tags: [tasks, background, ledger, taskflow, lifecycle, orchestration]
related:
  - "[[concepts/automation]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/cron]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/automation/tasks]]"
  - "[[sources/automation/taskflow]]"
updated: 2026-06-14
---

# タスク（Tasks）

タスク（tasks, 切り離して実行される作業を追跡する台帳）は、サブエージェント・ACP 実行・cron・メディア生成といった**非同期の仕事を一元的に追跡**する仕組み。複数ステップの耐久オーケストレーションは Task Flow がその上に重なる。

## なぜ重要か

エージェントが「裏で動かしておく作業」（動画生成、サブエージェントの調査、定期ジョブ）を持つと、「何が・いつ・どうなったか」を追える単一の真実が必要になる。タスク台帳（`openclaw tasks`）はそれを提供し、状態遷移・通知・監査・クリーンアップを通じて、切り離した作業を**見失わない**ようにする。これは [[concepts/automation]] の各仕組みが共有する基盤。

## 仕組み（要点）

- **作成源と既定通知**：ACP（`acp`）/サブエージェント（`subagent`、`sessions_spawn`）/cron（`silent`）/CLI/メディアジョブ（`music_generate`/`video_generate`）。
- **ライフサイクル**：`queued`→`running`→`succeeded`/`failed`/`timed_out`/`cancelled`/`lost`。
- **通知ポリシー**：`done_only`（既定）/`state_changes`/`silent`。監査：`stale_queued`/`stale_running`/`lost`/`delivery_failed`。
- **Task Flow**：複数タスクを耐久的に束ねるオーケストレーション（`managed`/`mirrored`、リビジョン追跡）。`openclaw tasks flow ...`。

## 既存 wiki とのつながり

タスクは [[concepts/multi-agent]] のサブエージェント生成、[[concepts/cron]] の各実行、メディア生成ツール（[[sources/tools/music-generation]]/[[sources/tools/video-generation]]）が作る非同期ジョブの共通台帳。外部からは Webhook Plugin（[[sources/plugins/webhooks]]）が TaskFlow を作り、オーサリングは [[sources/tools/lobster]]。[[components/gateway]] が台帳を保持する。

## 代表ソース

- [[sources/automation/tasks]] — 台帳・ライフサイクル・通知・監査
- [[sources/automation/taskflow]] — 複数ステップの耐久オーケストレーション

## 関連ページ

- [[concepts/automation]] / [[concepts/multi-agent]] / [[concepts/cron]]
- [[sources/plugins/webhooks]] / [[sources/tools/lobster]] / [[components/gateway]]
