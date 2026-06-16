---
type: concept
aliases: [Session tools, セッションツール, sessions_spawn]
tags: [session-tool, subagent, orchestration, visibility]
related:
  - "[[concepts/session]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/agent-loop]]"
sources:
  - "[[sources/concepts/session-tool]]"
updated: 2026-06-14
---

# セッションツール

エージェントが**セッションをまたいで作業し、サブエージェント（子エージェント）をオーケストレーションする**ためのツール群――`sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `sessions_yield` / `subagents` / `session_status`。[[concepts/session]] が「セッションとは何か」なら、本ページは「それを操作する道具」。

## なぜ重要か

1 つのエージェントに閉じない**並列・委任ワークフロー**を支える。`sessions_spawn` は分離した子を非ブロッキングで生成し（`runId`/`childSessionKey` を即返す）、`sessions_yield` で完了を push で待てる――ポーリングは避ける。どのツールが使えるかは [[concepts/agent]] のツールポリシー（`tools.profile: "coding"` は生成込み、`"messaging"` は送信のみ）が決め、**可視性スコープ**（`self`/`tree`(既定)/`agent`/`all`）が「どのセッションを見られるか」を制限する。サンドボックス化セッションは設定に関係なく `tree` に固定。

## 押さえる点

- 受信側はセッション間メッセージを `[Inter-session message ... isUser=false]` と**ツール経由データ**として扱う（ユーザーの直接指示にしない）。`sessions_history` は安全性フィルタ（XML/制御トークン除去・認証情報リダクト・切り詰め）付きの範囲制限ビュー。
- サブエージェントの実行系（`runtime: "subagent"`/`"acp"`、`context: "fork"`/`"isolated"`）は [[concepts/multi-agent]] と [[concepts/agent-runtimes]] につながる。

全ツールの詳細・生成オプション・スコープ表は [[sources/concepts/session-tool]]。

エージェントツール全体の選び分け（ツール/Skills/Plugins）は [[sources/tools/tools]]、コマンド実行の面は [[concepts/exec]]、Web 検索/取得は [[concepts/web-search]]、メディア生成/理解は [[sources/tools/media-overview]]。サブエージェント生成（`sessions_spawn`）の詳細は [[sources/tools/subagents]]、外部ハーネス委任は [[concepts/acp]]。個別ツール（[[sources/tools/diffs]]・[[sources/tools/apply-patch]]・[[sources/tools/tool-search]]・[[sources/tools/lobster]]・[[sources/tools/btw]]・[[sources/tools/trajectory]] 等）は `tools/` セクションの各ソースに。

## 関連

- [[concepts/session]] / [[concepts/multi-agent]] / [[concepts/agent-loop]]
- [[concepts/exec]] / [[concepts/skills]] / [[concepts/web-search]] / [[concepts/acp]]
- [[components/browser]] — Web ページを実際に操作する `browser` ツール（[[concepts/web-search]] が「読む」なら browser は「操作する」）
- [[sources/tools/tools]] / [[sources/tools/media-overview]] / [[sources/tools/subagents]]
