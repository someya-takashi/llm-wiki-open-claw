---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/exa-search
source_path: raw/docs/tools/exa-search.md
doc_section: tools
title: "Exa 検索"
ingested: 2026-06-14
tags: [web-search, provider, exa, neural-search, content-extraction]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Exa 検索（解説）

> 原典: `raw/docs/tools/exa-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/exa-search

## 一言まとめ

`web_search` の Exa プロバイダー。**ニューラル＋キーワード検索**に加え、コンテンツ抽出（ハイライト/テキスト/要約）に対応した構造化結果を返す。

## 位置づけ

[[concepts/web-search]] のプロバイダー（全体像は [[sources/tools/web]]）。意味的に近い文書を見つけたいときに有効。

## 設定・使い方の要点

- 認証：`EXA_API_KEY` または `plugins.entries.exa.config.webSearch.apiKey`（SecretRef 可）。`plugins.entries.exa.config.webSearch.baseUrl` でエンドポイント上書き。
- フィルター：ニューラル/キーワードモード・日付・コンテンツ抽出。自動検出順 65。

## 用語と略称

- **ニューラル検索** = 埋め込みで意味的に近い文書を探す検索
- **コンテンツ抽出** = ページ本文のハイライト/テキスト/要約取得
- **構造化結果** = 機械可読な検索結果

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/tavily]] / [[sources/tools/firecrawl]]
