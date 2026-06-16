---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/firecrawl
source_path: raw/docs/tools/firecrawl.md
doc_section: tools
title: "Firecrawl"
ingested: 2026-06-14
tags: [web-search, web-fetch, provider, firecrawl, scrape, extract]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web-fetch]]"
  - "[[sources/tools/web]]"
---

# Firecrawl（解説）

> 原典: `raw/docs/tools/firecrawl.md` ・ https://docs.openclaw.ai/ja-JP/tools/firecrawl

## 一言まとめ

Firecrawl は **Web 検索とスクレイピングの両方**を担うプロバイダー。`web_search` の構造化結果に加え、`firecrawl_search`/`firecrawl_scrape` で深い抽出、`web_fetch` の bot 回避フォールバックも提供する。

## 位置づけ

[[concepts/web-search]] のプロバイダーで、検索（[[sources/tools/web]]）と取得（[[sources/tools/web-fetch]]）の両面に効く唯一のバンドル。`web_fetch` の Readability 失敗時のフォールバック先。

## 設定・使い方の要点

- 認証：`FIRECRAWL_API_KEY` または `plugins.entries.firecrawl.config.{webSearch,webFetch}.apiKey`（SecretRef 可）。`webFetch`：`baseUrl`（ホスト型は `https://api.firecrawl.dev` 固定、セルフホストはプライベート対象のみ `http://` 可）・`onlyMainContent`・`maxAgeMs`。
- 深い抽出は専用ツール `firecrawl_search`/`firecrawl_scrape`（`web_search` 経由は `query`/`count` のみ）。自動検出順 60（検索）。

## 注意点・落とし穴

- ⚠️ Firecrawl 有効かつ SecretRef 未解決・env フォールバックも無いと **Gateway 起動が早期失敗**。

## 用語と略称

- **スクレイピング（scrape）** = ページから構造化データを抽出すること
- **`firecrawl_search` / `firecrawl_scrape`** = 検索/スクレイプの専用ツール
- **bot 回避** = ボット検知を回避して取得する仕組み
- **Readability フォールバック** = 抽出失敗時に Firecrawl で再試行

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]] / [[sources/tools/web-fetch]]
- [[sources/tools/exa-search]] / [[sources/tools/tavily]]
