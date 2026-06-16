---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/codex-computer-use
source_path: raw/docs/plugins/codex-computer-use.md
doc_section: plugins
title: "Codex コンピューター操作"
ingested: 2026-06-14
tags: [plugin, codex, computer-use, mcp, desktop-control, macos]
related:
  - "[[sources/plugins/codex-harness]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/sandboxing]]"
---

# Codex コンピューター操作（解説）

> 原典: `raw/docs/plugins/codex-computer-use.md` ・ https://docs.openclaw.ai/ja-JP/plugins/codex-computer-use

## 一言まとめ

Codex モードのエージェントに**ローカルデスクトップを操作させる** Codex ネイティブの `computer-use` MCP Plugin を準備する設定。OpenClaw はデスクトップアプリを提供せず操作も実行せず、Codex の権限を回避しない——「Codex に所有させる」段取りだけを担う。

## 位置づけ

[[sources/plugins/codex-harness]] が動いている前提の拡張で、[[concepts/agent-runtimes]] の Codex ランタイム上の機能。デスクトップ制御は [[concepts/sandboxing]]・[[concepts/threat-model]] 上は高リスク面（macOS 権限が必須）。

## 仕組み・ふるまい

- `plugins.entries.codex.config.computerUse`（`autoInstall`/`marketplaceSource`/`marketplacePath`/`marketplaceName`/`pluginName`/`mcpServerName`）。各 Codex モードターン前に app-server を確認し、必要なら Codex に `plugin/install`＋MCP 再読み込みを依頼。
- チャットコマンド `/codex computer-use status|install`（`status` は読み取り専用）。macOS は標準 Codex アプリの `openai-bundled` マーケットプレイスを自動登録試行。
- セットアップ理由コード：`disabled`/`marketplace_missing`/`plugin_not_installed`/`remote_install_unsupported`/`mcp_missing`/`ready` 等。

## 設定・使い方の要点

- ⚠️ `computerUse.enabled` が true なら**フェイルクローズ**（ネイティブツール無しでターンを暗黙続行しない）。設定変更後は対象チャットで `/new`/`/reset`。
- 別経路：`cua-driver mcp`（TryCua のドライバーを OpenClaw の MCP レジストリに直接登録、Codex マーケットプレイスを介さない）。

## 注意点・落とし穴

- リモート専用カタログは install 不可（ローカル `marketplaceSource`/`marketplacePath` が必要）。OpenClaw.app の Peekaboo ブリッジや iOS ノードは Computer Use とは**別物**。

## 用語と略称

- **Computer Use** = デスクトップを操作する Codex ネイティブ MCP 機能
- **MCP** = Model Context Protocol（外部ツール接続規格）
- **marketplace** = Codex の Plugin 配布元
- **cua-driver** = TryCua のデスクトップ制御ドライバー

## 関連ページ

- [[sources/plugins/codex-harness]] / [[sources/plugins/codex-native-plugins]]
- [[concepts/agent-runtimes]] / [[concepts/sandboxing]] / [[components/node]]
