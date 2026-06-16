---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/perplexity-search
source_path: raw/docs/tools/perplexity-search.md
doc_section: tools
title: "Perplexity 検索"
ingested: 2026-06-14
tags: [web-search, provider, perplexity, sonar, domain-filter, openrouter]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Perplexity 検索（解説）

> 原典: `raw/docs/tools/perplexity-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/perplexity-search

## 一言まとめ

`web_search` の Perplexity プロバイダー。**コンテンツ抽出制御とドメインフィルタ**を備えた構造化結果を返す。OpenRouter 経由の Sonar 互換パスにも対応。

## 位置づけ

[[concepts/web-search]] の構造化結果系プロバイダー（全体像は [[sources/tools/web]]）。フィルタの細かさが特徴。

## 設定・使い方の要点

- 認証：`PERPLEXITY_API_KEY`/`OPENROUTER_API_KEY` または `plugins.entries.perplexity.config.webSearch.apiKey`（SecretRef 可）。自動検出順 50。Sonar/OpenRouter 互換は `baseUrl`/`model` で。
- フィルター：`country`/`language`/`freshness`/`domain_filter`（許可/拒否、`-reddit.com` 等）・`max_tokens`（既定 25000）・`max_tokens_per_page`（既定 2048）。

## 用語と略称

- **Perplexity / Sonar** = Perplexity の検索ブランド/モデル
- **ドメインフィルタ** = 特定ドメインを含める/除外する
- **OpenRouter** = 複数モデルを束ねる API ゲートウェイ
- **コンテンツ抽出制御** = 取得本文量の制御（max_tokens）

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/tavily]] / [[sources/tools/exa-search]]
