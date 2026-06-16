---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/bundles
source_path: raw/docs/plugins/bundles.md
doc_section: plugins
title: "Plugin バンドル（Codex/Claude/Cursor）"
ingested: 2026-06-14
tags: [plugin, bundle, codex, claude, cursor, mcp, skills, lsp]
related:
  - "[[components/plugin-system]]"
  - "[[sources/tools/plugin]]"
  - "[[concepts/sandboxing]]"
---

# Plugin バンドル（Codex/Claude/Cursor）（解説）

> 原典: `raw/docs/plugins/bundles.md` ・ https://docs.openclaw.ai/ja-JP/plugins/bundles

## 一言まとめ

**Codex / Claude / Cursor** の 3 つの外部エコシステムの Plugin（＝バンドル）を、OpenClaw のネイティブ機能（Skills・フック・MCP ツール・LSP）にマッピングしてインストールする仕組み。ネイティブ Plugin と違い**プロセス内で任意コードを実行せず、信頼境界が狭い**コンテンツパック。

## 位置づけ

[[components/plugin-system]] の 2 形式のうち「バンドル」側（ネイティブ側は [[sources/tools/plugin]]）。狭い信頼境界は [[concepts/sandboxing]]・[[concepts/threat-model]] のサプライチェーン観点と整合。

## 仕組み・ふるまい

- **マッピング（現在サポート）**：Skill コンテンツ（バンドルの skill ルート＝通常 Skill、Claude `commands/`・Cursor `.cursor/commands/` も Skill 扱い）、フックパック（Codex の `HOOK.md`+`handler.ts`）、MCP ツール（組み込み Pi 設定に `mcpServers` としてマージ、stdio/HTTP）、LSP（Claude `.lsp.json`）、設定（Claude `settings.json`、シェル上書きキーはサニタイズ）。
- **MCP ツール名**：`serverName__toolName` のプロバイダー安全名に正規化（決定論的順序でキャッシュ安定）。
- **検出のみ（未実行）**：Claude の `agents`/`hooks.json`/`outputStyles`、Cursor の `.cursor/agents`/`hooks.json`/`rules`。
- **検出優先**：ネイティブ形式（`openclaw.plugin.json`/`openclaw.extensions`）を先に確認 → なければバンドルマーカー。両方あればネイティブ採用。

## 設定・使い方の要点

- `openclaw plugins install ./my-bundle[.tgz]` or marketplace（`marketplace list`/`install <name>@<marketplace>`）。`openclaw plugins inspect <id>` で `Format: bundle`＋サブタイプ（codex/claude/cursor）を確認、`gateway restart`。
- MCP トランスポート：stdio（`command`/`args`/`env`）or HTTP（`url`/`transport: streamable-http|sse`/`headers` は `${ENV}` 補間）。`coding`/`messaging` プロファイルは既定でバンドル MCP を含む（除外は `tools.deny: ["bundle-mcp"]`）。

## 注意点・落とし穴

- ⚠️ **バンドルはネイティブ Plugin と別物**：任意ランタイムをプロセス内で読み込まない。Skill/フックパスは Plugin ルート内に限定（境界チェック）。それでもサードパーティバンドルは「信頼するコンテンツ」として扱う。
- サードパーティ互換バンドルに起動時 `npm install` 修復は無い（必要物を同梱する）。`hooks/hooks.json` は検出のみ（実行可能フックは OpenClaw フックパックレイアウトかネイティブ Plugin で）。

## 用語と略称

- **バンドル（bundle）** = Codex/Claude/Cursor 形式のコンテンツパック
- **MCP** = Model Context Protocol（外部ツール接続規格）
- **LSP** = Language Server Protocol（エディタ言語支援）
- **stdio / streamable-http / sse** = MCP サーバーのトランスポート方式
- **Pi** = OpenClaw の組み込みエージェントハーネス

## 関連ページ

- [[components/plugin-system]] / [[sources/tools/plugin]]
- [[sources/plugins/codex-harness]] / [[concepts/sandboxing]] / [[concepts/threat-model]]
