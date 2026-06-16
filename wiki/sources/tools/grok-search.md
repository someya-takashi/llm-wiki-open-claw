---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/grok-search
source_path: raw/docs/tools/grok-search.md
doc_section: tools
title: "Grok 検索"
ingested: 2026-06-14
tags: [web-search, provider, grok, xai, ai-synthesis, x-search]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Grok 検索（解説）

> 原典: `raw/docs/tools/grok-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/grok-search

## 一言まとめ

`web_search` の Grok（xAI）プロバイダー。**xAI Web グラウンディング**で引用付きの AI 合成回答を返す。X 投稿検索 `x_search` と同じ xAI 認証/エンドポイントを共有する。

## 位置づけ

[[concepts/web-search]] の AI 合成系プロバイダー（全体像は [[sources/tools/web]]）。実体は同梱 `xai` Plugin（`x_search`・`code_execution` と同じ）。

## 設定・使い方の要点

- 認証：`XAI_API_KEY` または `plugins.entries.xai.config.webSearch.apiKey`（SecretRef 可）。自動検出順 30。
- `x_search` 設定（`plugins.entries.xai.config.xSearch.*`）と認証を共有。Grok 選択時に `x_search` セットアップも提示されうる。

## 用語と略称

- **xAI** = Grok を提供する企業（X 連携）
- **Web グラウンディング** = Web 検索に基づく回答生成
- **`x_search`** = X（Twitter）投稿検索ツール（同じ xAI Plugin）

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/gemini-search]] / [[sources/tools/code-execution]]
