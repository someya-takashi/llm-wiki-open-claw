---
type: concept
aliases: [Automation, 自動化, バックグラウンド作業]
tags: [automation, cron, heartbeat, commitments, tasks, hooks, standing-orders]
related:
  - "[[concepts/cron]]"
  - "[[concepts/heartbeat]]"
  - "[[concepts/commitments]]"
  - "[[concepts/tasks]]"
  - "[[concepts/hooks]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/automation/automation]]"
  - "[[sources/automation/standing-orders]]"
updated: 2026-06-14
---

# 自動化（Automation）

自動化（automation, ユーザーの即時メッセージなしにエージェントが作業を進める仕組み）は、OpenClaw が**バックグラウンドで動く**ための手段の総称。8 つの仕組みがあり、「正確なタイミングか / 文脈をどう持つか / 何をトリガーに動くか」で選び分ける。

## 8 つの仕組み

| 仕組み | いつ動くか | 概念/ソース |
|---|---|---|
| **Cron** | 正確な時刻・間隔・cron 式 | [[concepts/cron]] |
| **Heartbeat** | おおよそ定期（既定 30 分） | [[concepts/heartbeat]] |
| **Inferred Commitments** | 会話から推論されたフォローアップ | [[concepts/commitments]] |
| **Background Tasks** | 切り離した作業の追跡 | [[concepts/tasks]] |
| **Task Flow** | 複数ステップの耐久オーケストレーション | [[sources/automation/taskflow]] |
| **Hooks（内部）** | ライフサイクル/メッセージイベント | [[concepts/hooks]] |
| **Plugin hooks** | ツール呼び出し等のインプロセス介入 | [[sources/plugins/hooks]] |
| **Standing Orders** | 全セッションに常時注入 | [[sources/automation/standing-orders]] |

## 軸で捉える

- **タイミング**：Cron は正確（cron 式/単発）、Heartbeat はおおよそ。「9:00 ちょうどに」は Cron、「30 分ごとに受信箱」は Heartbeat。
- **セッション文脈**：Cron は新規/分離（クリーン）、Heartbeat はメインセッションの完全文脈。
- **トリガー種別**：時刻（Cron/Heartbeat）/ 推論（Commitments）/ イベント（Hooks）/ 常時（Standing Orders）。
- **記録**：Cron と Background Tasks は [[concepts/tasks]] のタスク記録を作る、Heartbeat は作らない。

## なぜ重要か

「人間のアシスタントのように、頼まれなくても気を利かせて動く」ためには、**いつ・どんな文脈で・何をきっかけに**自発実行するかを設計で分ける必要がある。OpenClaw はこれを 8 つの直交した仕組みに分解し、誤って全部を 1 つの手段でやろうとして壊れるのを防ぐ。これらはすべて [[components/gateway]] のプロセス内で駆動され、配信は [[concepts/messages]] を通る。

## 代表ソース

- [[sources/automation/automation]] — どの仕組みを選ぶかの判断ガイド
- [[sources/automation/standing-orders]] — 全セッション注入の常設指示

## 関連ページ

- [[concepts/cron]] / [[concepts/heartbeat]] / [[concepts/commitments]] / [[concepts/tasks]] / [[concepts/hooks]]
- [[concepts/dreaming]] / [[concepts/messages]] / [[components/gateway]]
