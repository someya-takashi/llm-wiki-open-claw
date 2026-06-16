---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/web-fetch
source_path: raw/docs/tools/web-fetch.md
doc_section: tools
title: "ウェブ取得（web_fetch）"
ingested: 2026-06-14
tags: [web-fetch, web_fetch, readability, ssrf, firecrawl, http-get]
related:
  - "[[concepts/web-search]]"
  - "[[concepts/security]]"
  - "[[concepts/session-tool]]"
---

# ウェブ取得（web_fetch）（解説）

> 原典: `raw/docs/tools/web-fetch.md` ・ https://docs.openclaw.ai/ja-JP/tools/web-fetch

## 一言まとめ

`web_fetch` は**指定 URL を HTTP GET し、読みやすいコンテンツ（HTML→markdown/text）を抽出**するツール。JavaScript は実行しない（JS 多用/ログインは [[sources/tools/browser]]）。既定で有効・設定不要。

## 位置づけ

[[concepts/web-search]] の取得側（検索は [[sources/tools/web]]）。SSRF 防御は [[concepts/security]]・[[sources/security/network-proxy]]、抽出失敗時のフォールバックは Firecrawl（[[sources/tools/firecrawl]]）。

## 仕組み・ふるまい

1. Chrome 風 UA で HTTP GET（プライベート/内部ホスト名をブロック、リダイレクト再チェック）→ 2. Readability でメインコンテンツ抽出 → 3. 失敗かつ Firecrawl 設定時は bot 回避モードで再試行 → 4. 同一 URL は 15 分キャッシュ。

## 設定・使い方の要点

- `tools.web.fetch.{enabled,provider,maxChars,maxResponseBytes,timeoutSeconds,maxRedirects,readability,userAgent}`。フォールバックプロバイダーは `tools.web.fetch.provider`（省略で自動検出、現在のバンドルは Firecrawl）。サンドボックス内取得はバンドルのみ。
- 信頼済み env プロキシ：`useTrustedEnvProxy: true` で DNS 解決をプロキシに委譲（ホスト名 SSRF チェックは維持）。

## 注意点・落とし穴

- ⚠️ **プライベート/内部ホスト名は遮断**。`ssrfPolicy.allowRfc2544BenchmarkRange`/`allowIpv6UniqueLocalRange` は信頼済み fake-IP プロキシ向けの狭いオプトイン（通常は未設定）。Readability 無効＋プロバイダー無しはフェイルクローズ。`web_fetch` はベストエフォート——一部サイトは [[sources/tools/browser]] が要る。

## 用語と略称

- **`web_fetch`** = URL を取得して本文抽出するツール
- **Readability** = HTML からメイン本文を抽出するライブラリ
- **SSRF** = Server-Side Request Forgery
- **fake-IP DNS** = プロキシが合成 IP を返す方式（Surge/Clash 等）
- **UA** = User-Agent（HTTP のクライアント識別ヘッダー）

## 関連ページ

- [[concepts/web-search]] / [[sources/tools/web]] / [[sources/tools/firecrawl]]
- [[concepts/security]] / [[sources/security/network-proxy]] / [[sources/tools/browser]]
