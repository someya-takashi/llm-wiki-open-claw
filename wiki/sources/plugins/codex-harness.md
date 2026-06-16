---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/codex-harness
source_path: raw/docs/plugins/codex-harness.md
doc_section: plugins
title: "Codex ハーネス"
ingested: 2026-06-14
tags: [plugin, codex, harness, agent-runtime, app-server, openai]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[components/plugin-system]]"
  - "[[sources/concepts/agent-runtimes]]"
---

# Codex ハーネス（解説）

> 原典: `raw/docs/plugins/codex-harness.md` ・ https://docs.openclaw.ai/ja-JP/plugins/codex-harness

## 一言まとめ

同梱の `codex` Plugin で、埋め込み OpenAI エージェントターンを**組み込み PI ハーネスでなく Codex app-server 経由**で実行する。ネイティブなスレッド再開・ツール継続・Compaction・app-server 実行を Codex に任せる「もう一つのエージェントランタイム」。

## 位置づけ

[[concepts/agent-runtimes]] の provider/model/runtime/channel の層分けにおける**ランタイム差し替え**そのもの（`openai/gpt-5.5` がモデル ref、`codex` がランタイム、チャネルは Telegram/Discord 等のまま）。Plugin としての実体は [[components/plugin-system]]。OpenClaw 側はチャネル・セッションファイル・モデル選択・OpenClaw 動的ツール・承認・メディア配信・表示トランスクリプトを担い続ける。

## 仕組み・ふるまい

- 通常は正規 OpenAI モデル ref（`openai/gpt-5.5`）を使い、`openai-codex/gpt-*` は使わない。認証順は `auth.order.openai`。
- Codex app-server スレッドを**コードモード専用**で開始し、遅延/検索可能な OpenClaw 動的ツールを Codex 自身のコード実行・ツール検索内に保持（PI 形式のラッパーを上乗せしない）。
- **app-server ポリシー**：承認・ツール継続・Compaction を Codex ネイティブで。`before_tool_call`/`after_tool_call` フックで Codex ネイティブツールに介入可能（[[sources/tools/plugin]] のフックブリッジ）。

## 設定・使い方の要点

- `plugins.entries.codex.enabled: true` ＋ provider/model に `agentRuntime.id: "codex"` か正規 `openai/*` ref。`openclaw plugins inspect codex --runtime` で検証。デプロイは基本 / 混在プロバイダー / **fail-closed**（Codex 不可なら停止）の 3 パターン。
- 拡張：[[sources/plugins/codex-computer-use]]（ローカルデスクトップ制御 MCP）、[[sources/plugins/codex-native-plugins]]（移行済みネイティブ Codex プラグイン）。

## 注意点・落とし穴

- Codex スレッドの調査は `/codex` 系コマンド（ランタイムコマンドで、`openclaw codex ...` CLI とは別）。ランタイム境界・v1 サポート契約は別リファレンス（codex-harness-runtime）。

## 用語と略称

- **ハーネス（harness）** = エージェントのターンを実行するバックエンド層
- **Codex app-server** = OpenAI Codex のローカル実行サーバー
- **PI** = OpenClaw の組み込みエージェントハーネス（既定）
- **コードモード** = Codex がコード実行でツールを扱うモード
- **fail-closed** = 要件未達なら通さず停止する安全側の挙動

## 関連ページ

- [[concepts/agent-runtimes]] — ランタイム層の概念
- [[sources/plugins/codex-computer-use]] / [[sources/plugins/codex-native-plugins]]
- [[components/plugin-system]] / [[sources/concepts/agent-runtimes]]
