---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/hooks
source_path: raw/docs/plugins/hooks.md
doc_section: plugins
title: "Plugin フック"
ingested: 2026-06-14
tags: [plugin, hooks, lifecycle, before-tool-call, llm-input, sdk]
related:
  - "[[components/plugin-system]]"
  - "[[sources/plugins/building-plugins]]"
  - "[[concepts/agent-loop]]"
---

# Plugin フック（解説）

> 原典: `raw/docs/plugins/hooks.md` ・ https://docs.openclaw.ai/ja-JP/plugins/hooks

## 一言まとめ

Plugin フックは、Plugin が**エージェント実行・ツール呼び出し・メッセージフロー・セッションライフサイクル・インストール・Gateway 起動を検査/変更する**インプロセス拡張点。`register(api)` の `on(...)`/`registerHook` で型付きライフサイクルフックに介入する。

## 位置づけ

[[components/plugin-system]] の動的な介入面で、[[concepts/agent-loop]] の各段にフックがかかる。オペレーターが `HOOK.md` スクリプトで `/new`/`gateway:startup` 等に反応する**内部フック（automation/hooks）とは別物**——こちらは Plugin の SDK フック。

## 仕組み・ふるまい

- **フックカタログ**：`before_model_resolve`（モデル解決前。モデル切替に推奨）/`before_agent_reply`/`before_agent_run`/`llm_input`/`llm_output`/`before_agent_finalize`/`agent_end`（会話フックは `entries.<id>.hooks.allowConversationAccess=true` が必要）、`before_tool_call`/`after_tool_call`（ツールポリシー）、`message_sending`、`before_install`、Gateway ライフサイクル。
- **決定セマンティクス**：`before_tool_call`/`before_install` の `{block: true}` は終端（低優先ハンドラをスキップ）、`{block: false}` は no-op（過去の block を解除しない）。`message_sending` は `{cancel: true}` が終端。
- ネイティブ Codex app-server はネイティブツールイベントをこのフック面にブリッジ（`before_tool_call` でブロック、`after_tool_call` で監視、`PermissionRequest` 承認に関与）。

## 設定・使い方の要点

- ツール結果の永続化・プロンプト/モデルフック・セッション拡張と次ターン注入・メッセージフック・インストールフックを用途別に使い分ける。完全な動作は SDK 概要（hook decision semantics）。

## 注意点・落とし穴

- ⚠️ 会話フック（`llm_input`/`llm_output` 等）は機密に触れるため明示オプトイン（`allowConversationAccess`）。`before_model_resolve` はモデル解決前に走るが `llm_output` はモデル出力後にのみ走る——切替には前者を使う。

## 用語と略称

- **フック（hook）** = ライフサイクルの特定点で実行される介入コールバック
- **`before_tool_call` / `after_tool_call`** = ツール実行の前後フック
- **`llm_input` / `llm_output`** = モデル入出力のフック
- **block / cancel** = 後続処理を止めるフック判定
- **内部フック（automation/hooks）** = オペレーターの `HOOK.md` スクリプト（別系統）

## 関連ページ

- [[components/plugin-system]] / [[sources/plugins/building-plugins]]
- [[concepts/agent-loop]] / [[concepts/sandboxing]] / [[sources/plugins/codex-harness]]
