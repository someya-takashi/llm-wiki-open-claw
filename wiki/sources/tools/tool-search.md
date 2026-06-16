---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/tool-search
source_path: raw/docs/tools/tool-search.md
doc_section: tools
title: "Tool Search"
ingested: 2026-06-14
tags: [tool-search, catalog, pi, experimental, dynamic-tools]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/context]]"
---

# Tool Search（解説）

> 原典: `raw/docs/tools/tool-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/tool-search

## 一言まとめ

Tool Search は OpenClaw PI エージェントの実験的機能で、**大規模ツールカタログをコンパクトに検索して呼び出す**仕組み。全ツールのスキーマをプロンプトに入れず、必要なものだけ動的に見つける。

## 位置づけ

[[concepts/session-tool]] のスケーリング手段で、[[concepts/context]] のトークン予算を守る（多数ツール時にスキーマ肥大を避ける）。PI ハーネス専用——Codex は独自のネイティブツール検索を使う（[[concepts/agent-runtimes]]）。

## 仕組み・ふるまい

- `tool_search_code`/`tool_search`/`tool_describe` で、実行時に利用可能な多数ツールから対象を検索・記述・呼び出し。`tools.toolSearch` で設定。

## 設定・使い方の要点

- 実行時ツールが多く、毎ターン少数しか使わない場合に有効。Codex ハーネス実行はこれに依存せず、ネイティブのコードモード/ツール検索/遅延動的ツールを使う。

## 注意点・落とし穴

- 実験的（PI 専用）。Codex ネイティブのツール検索とは別物。

## 用語と略称

- **Tool Search** = ツールカタログを検索して呼ぶ仕組み
- **PI** = OpenClaw の組み込みエージェントハーネス
- **動的ツール** = 実行時に解決されるツール
- **カタログ（catalog）** = 利用可能なツールの一覧

## 関連ページ

- [[concepts/session-tool]] / [[concepts/context]] / [[concepts/agent-runtimes]]
- [[sources/tools/tools]]
