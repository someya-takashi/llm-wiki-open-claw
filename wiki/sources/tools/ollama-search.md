---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/ollama-search
source_path: raw/docs/tools/ollama-search.md
doc_section: tools
title: "Ollama ウェブ検索"
ingested: 2026-06-14
tags: [web-search, provider, ollama, keyless, local]
related:
  - "[[concepts/web-search]]"
  - "[[sources/tools/web]]"
  - "[[concepts/local-models]]"
---

# Ollama ウェブ検索（解説）

> 原典: `raw/docs/tools/ollama-search.md` ・ https://docs.openclaw.ai/ja-JP/tools/ollama-search

## 一言まとめ

`web_search` の Ollama プロバイダー。サインイン済みのローカル Ollama ホスト、またはホスト型 Ollama API による検索で、構造化スニペットを返す。**キー不要フォールバック**の一つ。

## 位置づけ

[[concepts/web-search]] のキー不要系プロバイダー（全体像は [[sources/tools/web]]）。ローカル LLM 運用（[[concepts/local-models]]）と同じ Ollama を検索にも使える。

## 設定・使い方の要点

- サインイン済みローカルホストならキー不要（`ollama signin`）。ホストのベアラー認証を再利用可。直接 `https://ollama.com` 検索には `OLLAMA_API_KEY`。自動検出順 110。

## 用語と略称

- **Ollama** = ローカル LLM 実行ツール（検索 API も提供）
- **キー不要（keyless）** = サインイン済みなら API キー不要
- **ベアラー認証** = `Authorization: Bearer` トークン認証

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]] / [[concepts/local-models]]
- [[sources/tools/duckduckgo-search]] / [[sources/tools/searxng-search]]
