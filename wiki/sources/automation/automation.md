---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/automation
source_path: raw/docs/automation/automation.md
doc_section: automation
title: "自動化（概要）"
ingested: 2026-06-14
tags: [automation, cron, heartbeat, commitments, tasks, hooks, standing-orders]
related:
  - "[[concepts/automation]]"
  - "[[concepts/heartbeat]]"
  - "[[concepts/cron]]"
---

# 自動化（概要）（解説）

> 原典: `raw/docs/automation/automation.md` ・ https://docs.openclaw.ai/ja-JP/automation
>
> ℹ️ `automation/` セクションのランディング。**どの仕組みでバックグラウンド作業をするか**を選ぶための判断ガイド。

## 一言まとめ

OpenClaw がバックグラウンドで作業を実行する 8 つの仕組み——Scheduled Tasks（Cron）・Heartbeat・Inferred Commitments・Background Tasks・Task Flow・Hooks・Plugin hooks・Standing Orders——の比較と連携。

## 位置づけ

[[concepts/automation]] の中核ソース。各仕組みは [[concepts/cron]]・[[concepts/heartbeat]]・[[concepts/commitments]]・[[concepts/tasks]]・[[concepts/hooks]] と本セクションの standing-orders に対応する。

## 仕組み・ふるまい（どれを選ぶか）

| やりたいこと | 推奨 | 理由 |
|---|---|---|
| 毎日 9:00 にレポート送信 | **Cron** | 正確なタイミング・分離実行 |
| 20 分後にリマインド | **Cron**（`--at`） | 正確な単発 |
| 30 分ごとに受信箱確認 | **Heartbeat** | 他の確認とまとめ・文脈考慮 |
| 言及された面接の後に確認 | **Inferred Commitments** | メモリ的フォローアップ |
| 切り離した作業の状態を調べる | **Background Tasks** | タスク台帳が追跡 |
| 複数ステップ後に要約 | **Task Flow** | リビジョン追跡の耐久オーケストレーション |
| `/new` 等のライフサイクルで script | **Hooks** | イベント駆動 |
| 全ツール呼び出しでコード実行 | **Plugin hooks** | インプロセス介入（[[sources/plugins/hooks]]） |
| 返信前に常にチェック | **Standing Orders** | 全セッションに自動注入 |

- **Cron vs Heartbeat**：Cron は正確なタイミング・新規/分離セッション・常にタスク記録・チャネル/webhook/無音配信。Heartbeat はおおよそ（既定 30 分）・メインセッションの完全文脈・タスク記録なし・インライン配信。

## 設定・使い方の要点

- 連携：cron がメインセッション実行を予約 → Heartbeat ターンで処理、commitments/dreaming も Heartbeat が配信。Task Flow は tasks の上の耐久オーケストレーション。

## 用語と略称

- **Cron** = 時刻指定のスケジュール実行
- **Heartbeat** = メインセッションの定期エージェントターン
- **Inferred Commitments** = 会話から推論された短期フォローアップ
- **Task Flow** = 複数ステップの耐久ワークフロー
- **Standing Orders** = 全セッションに注入される常設指示

## 関連ページ

- [[concepts/automation]] — 対応する概念ページ
- [[sources/automation/cron-jobs]] / [[sources/automation/tasks]] / [[sources/automation/taskflow]] / [[sources/automation/hooks]] / [[sources/automation/standing-orders]]
- [[concepts/heartbeat]] / [[concepts/commitments]]
