---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/lobster
source_path: raw/docs/tools/lobster.md
doc_section: tools
title: "Lobster"
ingested: 2026-06-14
tags: [lobster, workflow, approval-checkpoints, deterministic, tool]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/tasks]]"
  - "[[components/plugin-system]]"
---

# Lobster（解説）

> 原典: `raw/docs/tools/lobster.md` ・ https://docs.openclaw.ai/ja-JP/tools/lobster

## 一言まとめ

Lobster は、複数ステップのツールシーケンスを**明示的な承認チェックポイント付きの単一で決定的な操作**として実行するワークフローシェル。

## 位置づけ

[[concepts/session-tool]] の Plugin ツールで、切り離しタスク（[[concepts/tasks]]）の一段上のオーサリングレイヤー。フローオーケストレーションは [[sources/automation/taskflow]]（`openclaw tasks flow`）、台帳は [[sources/automation/tasks]]。`.prose`（[[sources/prose]]）と並ぶワークフロー記述だが、Lobster は承認ゲート付きの決定論を強調。

## 仕組み・ふるまい

- 複数ツール呼び出しを 1 つの決定的フローにまとめ、要所に再開可能な承認チェックポイントを置く。各 LLM ステップは [[sources/tools/llm-task]] で差し込める。

## 設定・使い方の要点

- 承認チェックポイントで人間が確認してから次へ進む、再現性の高いワークフロー（コードレビュー/インシデントトリアージ等）に向く。

## 注意点・落とし穴

- OpenProse（非決定的・並列マルチエージェント）と Lobster（決定的・承認ゲート）は用途で使い分ける。

## 用語と略称

- **Lobster** = 承認チェックポイント付きの決定的ワークフローシェル
- **承認チェックポイント** = 次へ進む前に人間承認を挟む地点
- **ワークフローシェル** = 複数ステップを束ねる実行器
- **決定的（deterministic）** = 同じ入力で同じ手順を踏む性質

## 関連ページ

- [[concepts/session-tool]] / [[concepts/tasks]] / [[components/plugin-system]]
- [[sources/automation/taskflow]] / [[sources/tools/llm-task]] / [[sources/prose]]
