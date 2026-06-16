---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/gemini-search
source_path: raw/docs/tools/gemini-search.md
doc_section: tools
title: "Gemini 検索"
ingested: 2026-06-14
tags: [web-search, provider, gemini, ai-synthesis, grounding, citations]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Gemini 検索（解説）

> 原典: `raw/docs/tools/gemini-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/gemini-search

## 一言まとめ

`web_search` の Gemini プロバイダー。**Google Search グラウンディング**により、引用付きの **AI 合成回答**を返す（スニペット列挙でなく 1 つのまとまった回答）。

## 位置づけ

[[concepts/web-search]] の AI 合成系プロバイダー（Grok/Kimi と同型。全体像は [[sources/tools/web]]）。「調べて要約して」を 1 ツールで返したいときに有効。

## 設定・使い方の要点

- 認証：`plugins.entries.google.config.webSearch.apiKey`・`GEMINI_API_KEY`、低優先で `models.providers.google.apiKey`/`baseUrl` を再利用。自動検出順 20。
- `freshness`/`date_after`/`date_before` を Google Search の時間範囲に変換して対応。`count` は互換のため受理するが回答の形は変わらない。

## 用語と略称

- **グラウンディング（grounding）** = 検索結果に基づいて回答を生成
- **AI 合成回答** = 複数ソースをまとめた 1 つの回答（＋引用）
- **引用（citations）** = 回答の根拠となる出典リンク

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/grok-search]] / [[sources/tools/kimi-search]]
