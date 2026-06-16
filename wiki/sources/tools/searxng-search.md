---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/searxng-search
source_path: raw/docs/tools/searxng-search.md
doc_section: tools
title: "SearXNG 検索"
ingested: 2026-06-14
tags: [web-search, provider, searxng, self-hosted, meta-search]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# SearXNG 検索（解説）

> 原典: `raw/docs/tools/searxng-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/searxng-search

## 一言まとめ

`web_search` の **セルフホスト型メタ検索**プロバイダー。Google/Bing/DuckDuckGo などを集約し、構造化スニペットを返す。API キー不要（自前の SearXNG を立てる）。

## 位置づけ

[[concepts/web-search]] のセルフホスト系プロバイダー（全体像は [[sources/tools/web]]）。プライバシー重視・自前運用したいときの選択肢。

## 設定・使い方の要点

- `SEARXNG_BASE_URL` または `plugins.entries.searxng.config.webSearch.baseUrl`。自動検出順 200（最後）。フィルター：カテゴリ・言語。
- ⚠️ `http://` は信頼できるプライベート/loopback ホストのみ。公開エンドポイントは `https://` 必須。

## 用語と略称

- **SearXNG** = オープンソースのメタ検索エンジン
- **メタ検索** = 複数検索エンジンの結果を集約する検索
- **セルフホスト** = 自前サーバーで運用すること

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/duckduckgo-search]] / [[sources/tools/ollama-search]]
