---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/duckduckgo-search
source_path: raw/docs/tools/duckduckgo-search.md
doc_section: tools
title: "DuckDuckGo 検索"
ingested: 2026-06-14
tags: [web-search, provider, duckduckgo, keyless, fallback]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# DuckDuckGo 検索（解説）

> 原典: `raw/docs/tools/duckduckgo-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/duckduckgo-search

## 一言まとめ

`web_search` の **キー不要フォールバック**プロバイダー。API キー不要の非公式 HTML ベース統合で、構造化スニペットを返す。自動検出ではキー不要系の先頭（順序 100）。

## 位置づけ

[[concepts/web-search]] のプロバイダー（全体像は [[sources/tools/web]]）。API キーを用意せずに検索を有効化する最小構成として有用。

## 設定・使い方の要点

- 認証不要。`tools.web.search.provider: "duckduckgo"` で明示選択、または自動検出のフォールバックとして。フィルターは限定的。

## 注意点・落とし穴

- 非公式 HTML スクレイピングベースのため、安定性は公式 API プロバイダー（Brave/Tavily 等）に劣る。本格運用は API 系を推奨。

## 用語と略称

- **キー不要（keyless）** = API キー不要で使えるプロバイダー
- **フォールバック** = 他が未設定のときに使う代替
- **HTML ベース統合** = 非公式に HTML を解析する方式

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/searxng-search]] / [[sources/tools/ollama-search]]
