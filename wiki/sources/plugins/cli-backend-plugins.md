---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/cli-backend-plugins
source_path: raw/docs/plugins/cli-backend-plugins.md
doc_section: plugins
title: "CLI バックエンド Plugin の構築"
ingested: 2026-06-14
tags: [plugin, sdk, cli-backend, fallback, mcp-bridge, provider-prefix]
related:
  - "[[components/plugin-system]]"
  - "[[concepts/agent-runtimes]]"
  - "[[sources/gateway/cli-backends]]"
---

# CLI バックエンド Plugin の構築（解説）

> 原典: `raw/docs/plugins/cli-backend-plugins.md` ・ https://docs.openclaw.ai/ja-JP/plugins/cli-backend-plugins

## 一言まとめ

ローカルの AI CLI を **テキスト推論バックエンド**として OpenClaw に登録する Plugin の作り方。バックエンドはモデル参照のプロバイダープレフィックス（例 `acme-cli/acme-large`）として現れる。

## 位置づけ

[[components/plugin-system]] で CLI バックエンドを登録する開発ガイドで、[[concepts/agent-runtimes]] のテキスト専用フォールバック層（同梱 claude-cli/codex は [[sources/gateway/cli-backends]]）を**自作する**側。

## 仕組み・ふるまい

- **Plugin が所有するもの**：CLI の起動・入出力（args/output 形式 json/jsonl/text・input arg/stdin）・セッション継続・画像パススルー。`api.registerCliBackend(...)` で登録。
- **設定の形**：[[sources/gateway/cli-backends]] と同じ `agents.defaults.cliBackends.<id>` フィールド（command/modelArg/sessionArg/output 等）。高度なバックエンドフック・**MCP ツールブリッジ**（`bundleMcp: true` で Gateway ツールを CLI に公開）。

## 設定・使い方の要点

- 検証：`openclaw agent --model <provider>/<model>` で動作確認、チェックリストに沿って output 解析・セッション・画像を確認。

## 注意点・落とし穴

- CLI バックエンドは ACP ではない（完全なハーネス制御が要るなら ACP）。所有 Plugin が `/think` の argv マッパーを宣言しないと `/think` は CLI に効かない。

## 用語と略称

- **CLI バックエンド** = ローカル AI CLI をテキスト推論に使う仕組み
- **プロバイダープレフィックス** = モデル参照左側のバックエンド ID（`acme-cli/...`）
- **MCP ツールブリッジ** = `bundleMcp` で Gateway ツールを CLI に公開する仕組み
- **JSONL** = JSON Lines（ストリーム出力形式）

## 関連ページ

- [[components/plugin-system]] / [[concepts/agent-runtimes]] / [[sources/gateway/cli-backends]]
- [[sources/plugins/building-plugins]]
