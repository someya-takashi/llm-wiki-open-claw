---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/minimax-search
source_path: raw/docs/tools/minimax-search.md
doc_section: tools
title: "MiniMax 検索"
ingested: 2026-06-14
tags: [web-search, provider, minimax, structured-results, region]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# MiniMax 検索（解説）

> 原典: `raw/docs/tools/minimax-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/minimax-search

## 一言まとめ

`web_search` の MiniMax プロバイダー。MiniMax Token Plan の検索 API で**構造化スニペット**を返し、リージョン（`global`/`cn`）フィルターに対応。

## 位置づけ

[[concepts/web-search]] の構造化結果系プロバイダー（全体像は [[sources/tools/web]]）。MiniMax は TTS/動画生成でも使うブランド。

## 設定・使い方の要点

- 認証：`MINIMAX_CODE_PLAN_KEY`/`MINIMAX_CODING_API_KEY`/`MINIMAX_OAUTH_TOKEN`/`MINIMAX_API_KEY` または `plugins.entries.minimax.config.webSearch.apiKey`。自動検出順 15（API 系で 2 番目）。

## 用語と略称

- **MiniMax Token Plan** = MiniMax の課金プラン
- **リージョン（region）** = 検索対象の地域（global/cn）
- **構造化スニペット** = タイトル/URL/抜粋の検索結果

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/brave-search]] / [[sources/tools/perplexity-search]]
