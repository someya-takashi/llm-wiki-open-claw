---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/automation/tasks
source_path: raw/docs/automation/tasks.md
doc_section: automation
title: "バックグラウンドタスク"
ingested: 2026-06-14
tags: [tasks, background, ledger, lifecycle, notify-policy, audit]
related:
  - "[[concepts/tasks]]"
  - "[[concepts/automation]]"
  - "[[concepts/multi-agent]]"
---

# バックグラウンドタスク（解説）

> 原典: `raw/docs/automation/tasks.md` ・ https://docs.openclaw.ai/ja-JP/automation/tasks

## 一言まとめ

切り離された作業（サブエージェント・ACP 実行・cron・メディア生成）を追跡する**タスク台帳**。`openclaw tasks` で一覧・調査・キャンセル・通知・監査でき、各タスクのライフサイクルと通知ポリシーを管理する。

## 位置づけ

[[concepts/tasks]] の中核ソース。[[concepts/multi-agent]] のサブエージェント生成（`sessions_spawn`）・[[concepts/cron]]・[[sources/tools/music-generation]]/[[sources/tools/video-generation]] が作る非同期ジョブの共通台帳。複数ステップの耐久オーケストレーションは [[sources/automation/taskflow]]。

## 仕組み・ふるまい

- **作成源**：ACP バックグラウンド実行（`acp`）/サブエージェント（`subagent`）/cron（`cron`）/CLI（`cli`）/メディアジョブ（`music_generate`/`video_generate`）。それぞれ既定通知ポリシーを持つ。
- **ライフサイクル**：`queued`→`running`→`succeeded`/`failed`/`timed_out`/`cancelled`/`lost`（5 分猶予後に裏付け状態喪失）。
- **通知ポリシー**：`done_only`（既定、終端のみ）/`state_changes`（全遷移）/`silent`。

## 設定・使い方の要点

- CLI：`openclaw tasks list [--filter]`/`show`/`cancel`/`audit`/`cleanup`。監査検出：`stale_queued`（10 分超キュー、warn）/`stale_running`（30 分超実行、error）/`lost`/`delivery_failed`/`missing_cleanup`。

## 注意点・落とし穴

- `lost` は保持中は warn、`cleanupAfter` 後に error。配信失敗（`delivery_failed`）は通知ポリシーが `silent` 以外のとき警告。

## 用語と略称

- **タスク台帳（task ledger）** = 切り離された作業を追跡する記録
- **ACP** = Agent Client Protocol（外部ハーネス制御）
- **`sessions_spawn`** = サブエージェントを生成するツール
- **通知ポリシー（notify policy）** = どの状態変化を配信するか

## 関連ページ

- [[concepts/tasks]] — 対応する概念ページ
- [[sources/automation/taskflow]] / [[concepts/automation]] / [[concepts/multi-agent]]
- [[sources/tools/lobster]] / [[sources/tools/music-generation]]
