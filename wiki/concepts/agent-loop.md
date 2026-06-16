---
type: concept
aliases: [Agent loop, エージェントループ, agentic loop]
tags: [agent-loop, session, queue, hooks, streaming, timeout]
related:
  - "[[concepts/agent]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/context]]"
  - "[[concepts/queue]]"
  - "[[concepts/compaction]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/agent-loop]]"
updated: 2026-06-14
---

# エージェントループ

**エージェントループ**は、受信メッセージを「アクション＋最終返信」に変える OpenClaw の**権威ある実行パス**である：受付 → コンテキスト組立 → モデル推論 → ツール実行 → ストリーミング返信 → 永続化。これがセッションごとに **1 本の直列化された実行**として走り、その間に lifecycle / assistant / tool のイベントを発行する。

## なぜ重要か

エージェントの「正しさ」と「壊れにくさ」がここに集約される。**セッションレーンでの直列化**がツール／履歴の競合を防ぎ、**セッション書き込みロック**がトランスクリプトの一貫性を守る。`agent` RPC が即 ack（`{runId, acceptedAt}`）を返し実行は非同期、という形が、ストリーミングや `agent.wait`・タイムアウト・キャンセルといった運用挙動の土台になっている。フックポイント（`before_model_resolve`・`before_tool_call`・`agent_end` ほか）がこのパス上に並ぶため、**ループを理解することが Plugin 拡張の前提**になる。

## 位置づけ

- 起点は [[components/gateway]] の `agent` RPC（または [[components/cli]] の `agent` コマンド）。
- [[concepts/agent]] が定めるワークスペース・セッション契約の上で走る。
- 実行を担うバックエンドは [[concepts/agent-runtimes]] が決める（PI 埋め込みか Codex app-server かで所有権が変わる）。
- 供給するキューモード（collect/steer/followup）は [[concepts/queue]]、長い履歴の圧縮は [[concepts/compaction]]。

フローの全体図・フック一覧・タイムアウトの実値は解説ページ [[sources/concepts/agent-loop]] を参照。

## 関連

- [[concepts/agent]] / [[concepts/agent-runtimes]]
- [[concepts/compaction]] / [[concepts/queue]] / [[concepts/streaming]]
