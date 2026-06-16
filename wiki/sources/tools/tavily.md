---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/tavily
source_path: raw/docs/tools/tavily.md
doc_section: tools
title: "Tavily"
ingested: 2026-06-14
tags: [web-search, provider, tavily, search-depth, extract]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Tavily（解説）

> 原典: `raw/docs/tools/tavily.md` ・ https://docs.openclaw.ai/ja-JP/tools/tavily

## 一言まとめ

`web_search` の Tavily プロバイダー。**検索深度・トピックフィルタ**に加え、URL 抽出用の `tavily_extract` を備えた構造化結果を返す。

## 位置づけ

[[concepts/web-search]] の構造化結果系プロバイダー（全体像は [[sources/tools/web]]）。深い抽出は専用ツールで行う。

## 設定・使い方の要点

- 認証：`TAVILY_API_KEY` または `plugins.entries.tavily.config.webSearch.apiKey`（SecretRef 可）。自動検出順 70。
- `web_search` 経由は `query`/`count` のみ。高度なオプションは専用ツール `tavily_search`/`tavily_extract`。

## 用語と略称

- **Tavily** = LLM 向けに最適化した検索 API
- **検索深度（search depth）** = 取得する詳細度の設定
- **`tavily_extract`** = URL から本文を抽出する専用ツール

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/exa-search]] / [[sources/tools/firecrawl]]
