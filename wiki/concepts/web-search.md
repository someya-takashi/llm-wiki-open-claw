---
type: concept
aliases: [Web Search, ウェブ検索, web_search, web_fetch, Web Tools]
tags: [web-search, web_search, web_fetch, x_search, providers, ssrf]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/security]]"
  - "[[concepts/agent-runtimes]]"
components:
  - "[[components/plugin-system]]"
sources:
  - "[[sources/tools/web]]"
  - "[[sources/tools/web-fetch]]"
updated: 2026-06-14
---

# ウェブ検索（Web Search）

ウェブ検索（web search, エージェントが Web から情報を取る道具）は、3 つのツール——`web_search`（検索）・`x_search`（X 投稿検索）・`web_fetch`（特定 URL 取得）——と、その背後の**差し替え可能なプロバイダー群**から成る。いずれも軽量 HTTP ツールで、JS 多用サイトは [[sources/tools/browser]] の領分。

## 3 つのツールと使い分け

| ツール | 何をするか | プロバイダー |
|---|---|---|
| **`web_search`** | キーワードで Web を検索 | 12 種（下表） |
| **`x_search`** | X（Twitter）投稿を検索 | xAI |
| **`web_fetch`** | 既知 URL を取得して本文抽出 | ローカル（Readability）＋Firecrawl フォールバック |

## プロバイダー（差し替え可能）

`web_search` は **12 のプロバイダー**から選べる。結果は 2 系統——**構造化スニペット**（[[sources/tools/brave-search]]・[[sources/tools/duckduckgo-search]]・[[sources/tools/exa-search]]・[[sources/tools/firecrawl]]・[[sources/tools/minimax-search]]・[[sources/tools/ollama-search]]・[[sources/tools/perplexity-search]]・[[sources/tools/searxng-search]]・[[sources/tools/tavily]]）と **AI 合成＋引用**（[[sources/tools/gemini-search]]・[[sources/tools/grok-search]]・[[sources/tools/kimi-search]]）。各プロバイダーは [[components/plugin-system]] の Plugin で、`provider` 未設定なら認証情報から自動検出（API 系優先 → キー不要系 → Brave フォールバック）。

## なぜ重要か

エージェントが「今の情報」を扱うには Web アクセスが要るが、これは**最大の SSRF 面**でもある。OpenClaw は `web_search`/`web_fetch` をガード付きフェッチに通し、プライベート/loopback/メタデータ宛先を遮断する（[[concepts/security]]・[[sources/security/network-proxy]]）。プロバイダーを差し替え可能にすることで、無料の DuckDuckGo から引用付きの Perplexity/Gemini まで、用途とコストに応じて選べる。OpenAI/Codex モデルはプロバイダーネイティブの検索を使うこともある（[[concepts/agent-runtimes]]）。

## 既存 wiki とのつながり

検索ツールは [[concepts/session-tool]] のツール群の一部（許可は `group:web`）。`web_fetch` の抽出失敗時は Firecrawl にフォールバックし、深いスクレイピングは `firecrawl_search`/`tavily_extract` 等の専用ツールで行う。プロバイダーキーは SecretRef（[[concepts/secrets]]）で外部化できる。

## 代表ソース

- [[sources/tools/web]] — web_search/x_search の概要・プロバイダー比較・自動検出
- [[sources/tools/web-fetch]] — URL 取得・Readability・SSRF・Firecrawl フォールバック

## 関連ページ

- [[concepts/session-tool]] / [[concepts/security]] / [[concepts/agent-runtimes]]
- [[components/plugin-system]] / [[sources/tools/browser]] / [[concepts/secrets]]
