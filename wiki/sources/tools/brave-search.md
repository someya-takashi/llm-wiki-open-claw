---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/brave-search
source_path: raw/docs/tools/brave-search.md
doc_section: tools
title: "Brave 検索"
ingested: 2026-06-14
tags: [web-search, provider, brave, structured-results]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Brave 検索（解説）

> 原典: `raw/docs/tools/brave-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/brave-search

## 一言まとめ

`web_search` の Brave プロバイダー。スニペット付きの**構造化結果**を返し、`llm-context` モード・国/言語/時間フィルターに対応。無料枠あり。自動検出の最優先（順序 10）かつ最終フォールバック先。

## 位置づけ

[[concepts/web-search]] のプロバイダーバックエンドの一つ（全体像は [[sources/tools/web]]）。実体は同梱 `brave` Plugin。

## 設定・使い方の要点

- 認証：`BRAVE_API_KEY` または `plugins.entries.brave.config.webSearch.apiKey`（SecretRef 可）。
- フィルター：`country`/`language`/`search_lang`/`ui_lang`/`freshness`/`date_after`+`date_before`、`llm-context` モード。
- ⚠️ `llm-context` モードは `ui_lang` を拒否。カスタム鮮度は `date_after` と `date_before` の両方が必要。

## 用語と略称

- **構造化結果** = タイトル/URL/スニペットの機械可読な検索結果
- **`llm-context` モード** = LLM 向けに整形した結果モード
- **freshness** = 時間範囲フィルター（day/week/month/year）

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/duckduckgo-search]] / [[sources/tools/perplexity-search]]
