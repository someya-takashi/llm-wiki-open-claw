---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/web
source_path: raw/docs/tools/web.md
doc_section: tools
title: "ウェブ検索（web_search / x_search / web_fetch）"
ingested: 2026-06-14
tags: [web-search, web_search, x_search, providers, auto-detect, ssrf]
related:
  - "[[concepts/web-search]]"
  - "[[concepts/session-tool]]"
  - "[[components/plugin-system]]"
---

# ウェブ検索（web_search / x_search / web_fetch）（解説）

> 原典: `raw/docs/tools/web.md` ・ https://docs.openclaw.ai/ja-JP/tools/web

## 一言まとめ

`web_search`（Web 検索）・`x_search`（X 投稿検索）・`web_fetch`（URL 取得）の概要と、**12 の検索プロバイダー**から選ぶ仕組み。`web_search` は軽量 HTTP ツールでブラウザ自動化ではない（JS 多用サイトは [[sources/tools/browser]]）。

## 位置づけ

[[concepts/web-search]] の中核ソース。各プロバイダーは [[components/plugin-system]] の Plugin として登録され、[[concepts/session-tool]] のツールとして現れる。特定 URL 取得は [[sources/tools/web-fetch]]。

## 仕組み・ふるまい

- **2 系統の結果形式**：構造化スニペット（Brave/DDG/Exa/Firecrawl/MiniMax/Ollama/Perplexity/SearXNG/Tavily）と **AI 合成＋引用**（Gemini/Grok/Kimi）。結果はクエリごと 15 分キャッシュ。
- **自動検出順**（`provider` 未設定時）：API 系 Brave→MiniMax→Gemini→Grok→Kimi→Perplexity→Firecrawl→Exa→Tavily、キー不要系 DuckDuckGo→Ollama→SearXNG、最後に Brave フォールバック。
- **ネイティブ検索**：OpenAI 直モデルはホスト型 `web_search` を自動使用、Codex は `tools.web.search.openaiCodex` でネイティブ Responses 検索。

## 設定・使い方の要点

- `tools.web.search.{enabled,provider,maxResults,cacheTtlMinutes}`。プロバイダー固有は `plugins.entries.<plugin>.config.webSearch.*`（API キーは SecretRef 可）。`openclaw configure --section web`。
- パラメーター：`query`（必須）・`count`・`country`・`language`・`freshness`・`date_after/before`・`domain_filter`（Perplexity）。`x_search` は `plugins.entries.xai.config.xSearch.*`（ハンドル/日付フィルター・画像/動画理解）。ツール許可は `group:web`（web_search/x_search/web_fetch を含む）。

## 注意点・落とし穴

- ⚠️ 管理対象 `web_search` はガード付きフェッチ（信頼済みプロバイダーホストのみ fake-IP DNS を限定許可、他のプライベート/loopback/メタデータは遮断）。プロバイダー ID のタイプミスは検証で失敗（暗黙フォールバックしない）。

## 用語と略称

- **`web_search` / `x_search` / `web_fetch`** = Web 検索 / X 投稿検索 / URL 取得
- **グラウンディング（grounding）** = 検索結果に基づいて回答を生成すること
- **SSRF** = Server-Side Request Forgery（内部宛先への不正リクエスト）
- **SecretRef** = env/file/exec から秘密を解決する参照
- **`group:web`** = web 系ツールをまとめた許可リストグループ

## 関連ページ

- [[concepts/web-search]] — 対応する概念ページ
- [[sources/tools/web-fetch]] / [[sources/tools/browser]] / [[concepts/session-tool]]
- プロバイダー：[[sources/tools/brave-search]] / [[sources/tools/perplexity-search]] / [[sources/tools/tavily]] ほか
