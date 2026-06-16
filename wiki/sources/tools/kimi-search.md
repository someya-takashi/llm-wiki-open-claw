---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/kimi-search
source_path: raw/docs/tools/kimi-search.md
doc_section: tools
title: "Kimi 検索"
ingested: 2026-06-14
tags: [web-search, provider, kimi, moonshot, ai-synthesis]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
---

# Kimi 検索（解説）

> 原典: `raw/docs/tools/kimi-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/kimi-search

## 一言まとめ

`web_search` の Kimi（Moonshot）プロバイダー。**Moonshot Web 検索**で引用付きの AI 合成回答を返す。グラウンディングされていないチャットへのフォールバックは**明示的に失敗**する（出典のない回答を出さない）。

## 位置づけ

[[concepts/web-search]] の AI 合成系プロバイダー（全体像は [[sources/tools/web]]）。

## 設定・使い方の要点

- 認証：`KIMI_API_KEY`/`MOONSHOT_API_KEY` または `plugins.entries.moonshot.config.webSearch.apiKey`。自動検出順 40。
- セットアップで Moonshot リージョン（`api.moonshot.ai` / `api.moonshot.cn`）と既定モデル（既定 `kimi-k2.6`）を尋ねられる。

## 注意点・落とし穴

- ⚠️ グラウンディング失敗時は通常チャットにフォールバックせず**失敗する**——根拠のない回答を返さない設計。

## 用語と略称

- **Kimi / Moonshot** = Moonshot AI の検索/モデルブランド
- **グラウンディング** = 検索結果に基づく回答
- **フォールバック失敗** = 根拠が取れないとき意図的にエラーにする挙動

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]]
- [[sources/tools/gemini-search]] / [[sources/tools/grok-search]]
