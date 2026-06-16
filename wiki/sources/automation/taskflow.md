---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/automation/taskflow
source_path: raw/docs/automation/taskflow.md
doc_section: automation
title: "タスクフロー（Task Flow）"
ingested: 2026-06-14
tags: [taskflow, managed-flow, mirrored, orchestration, revision-tracking]
related:
  - "[[concepts/tasks]]"
  - "[[sources/automation/tasks]]"
  - "[[concepts/automation]]"
---

# タスクフロー（Task Flow）（解説）

> 原典: `raw/docs/automation/taskflow.md` ・ https://docs.openclaw.ai/ja-JP/automation/taskflow

## 一言まとめ

複数ステップ（A→B→C）のバックグラウンド作業を**耐久的にオーケストレーション**する仕組み。リビジョン追跡付きで、フロー全体の状態とキャンセルを管理する（単発ジョブは通常タスク、単発リマインダーは cron）。

## 位置づけ

[[concepts/tasks]] の上位レイヤー（個別タスク台帳は [[sources/automation/tasks]]）。Webhook Plugin（[[sources/plugins/webhooks]]）が外部から TaskFlow を作る入口になり、オーサリングは [[sources/tools/lobster]]。

## 仕組み・ふるまい

- **同期モード**：`managed`（OpenClaw が作成・進行を所有）/`mirrored`（外部で作られたタスクを監視）。
- **耐久状態とリビジョン追跡**：フローの状態は永続化され、改訂を追える。キャンセルは実行中フローとアクティブタスクを止める。
- フローとタスクは親子関係（フローが複数の管理対象タスクを束ねる）。

## 設定・使い方の要点

- CLI：`openclaw tasks flow list`（ステータス＋同期モード）/`flow show <id>`/`flow cancel <id>`。

## 注意点・落とし穴

- 「単一ジョブなら通常タスク、パイプラインなら Task Flow、外部作成の監視は mirrored、単発リマインダーは cron」と使い分ける。

## 用語と略称

- **Task Flow** = 複数ステップの耐久ワークフロー
- **managed / mirrored** = 自前で所有 / 外部タスクを監視
- **リビジョン追跡** = フロー状態の改訂履歴
- **オーケストレーション** = 複数タスクの順序・依存の制御

## 関連ページ

- [[concepts/tasks]] / [[sources/automation/tasks]] / [[concepts/automation]]
- [[sources/tools/lobster]] / [[sources/plugins/webhooks]]
