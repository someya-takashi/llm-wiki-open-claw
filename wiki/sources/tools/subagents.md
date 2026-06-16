---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/subagents
source_path: raw/docs/tools/subagents.md
doc_section: tools
title: "サブエージェント"
ingested: 2026-06-14
tags: [subagents, sessions_spawn, delegation, background, multi-agent]
related:
  - "[[concepts/multi-agent]]"
  - "[[concepts/session-tool]]"
  - "[[concepts/tasks]]"
---

# サブエージェント（解説）

> 原典: `raw/docs/tools/subagents.md` ・ https://docs.openclaw.ai/ja-JP/tools/subagents

## 一言まとめ

サブエージェントは、既存のエージェント実行から**生成されるバックグラウンドのエージェント実行**。それぞれ専用セッション（`agent:<agentId>:subagent:<uuid>`）で走り、完了すると結果を呼び出し元チャネルへ通知する。各実行は [[concepts/tasks]] のタスクとして追跡。

## 位置づけ

[[concepts/multi-agent]] の委任実行の道具で、[[concepts/session-tool]] の `sessions_spawn`/`sessions_yield`/`subagents` ツール群。ACP ランタイムで外部ハーネスに委任する場合は [[concepts/acp]]。

## 仕組み・ふるまい

- **`sessions_spawn`**：サブエージェントを生成。委任プロンプトモード・タスク名/ターゲット指定・コンテキストモード（`fork`/`isolated`）。`runtime: "subagent"`/`"acp"`。
- **`sessions_yield`**：親に制御/結果を返す。**`subagents`**：実行中サブエージェントの管理（list/kill/log/info/send/steer）。
- スラッシュコマンド `/subagents`・スレッドバインディング制御。スレッド対応チャネル（Discord スレッド/Telegram トピック）にバインド可。

## 設定・使い方の要点

- 並列調査や専門タスクの委任に使う（[[concepts/parallel-specialist-lanes]]）。各サブエージェントは独自セッション＝コンテキスト分離。

## 注意点・落とし穴

- サブエージェントはバックグラウンド実行——進捗/結果は [[sources/automation/tasks]] の台帳と通知ポリシーで追う。実行中への誘導は [[sources/tools/steer]]。

## 用語と略称

- **サブエージェント（subagent）** = 親実行から生成された子エージェント実行
- **`sessions_spawn` / `sessions_yield`** = 子生成 / 親への返却ツール
- **コンテキストモード** = `fork`（親の文脈を引き継ぐ）/`isolated`（分離）
- **スレッドバインディング** = Discord スレッド等にセッションを束ねる

## 関連ページ

- [[concepts/multi-agent]] / [[concepts/session-tool]] / [[concepts/tasks]]
- [[concepts/acp]] / [[concepts/parallel-specialist-lanes]] / [[sources/tools/steer]]
