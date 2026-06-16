---
title: "ウェブ取得"
source: "https://docs.openclaw.ai/ja-JP/tools/web-fetch"
author:
published:
created: 2026-06-14
description: "web_fetch ツール -- 読みやすいコンテンツ抽出付きの HTTP フェッチ"
tags:
  - "clippings"
---
`web_fetch` ツールは通常の HTTP GET を実行し、読み取り可能なコンテンツを抽出します （HTML から markdown または text）。JavaScript は実行 **しません** 。

JS が多用されているサイトやログイン保護されたページには、代わりに [Webブラウザー](https://docs.openclaw.ai/ja-JP/tools/browser) を使用してください。

## クイックスタート

`web_fetch` は **デフォルトで有効** です -- 設定は不要です。エージェントは すぐに呼び出せます。

javascript

```javascript
await web_fetch({ url: "https://example.com/article" });
```

## ツールパラメーター

取得する URL。 `http(s)` のみ。

メインコンテンツ抽出後の出力形式。

出力をこの文字数に切り詰めます。

## 仕組み

- ### Fetch
	Chrome 風の User-Agent と `Accept-Language` ヘッダーで HTTP GET を送信します。 プライベート/内部ホスト名をブロックし、リダイレクトを再チェックします。
- ### Extract
	HTML レスポンスに対して Readability（メインコンテンツ抽出）を実行します。
- ### Fallback (optional)
	Readability が失敗し、Firecrawl が設定されている場合は、bot 回避モードで Firecrawl API 経由で再試行します。
- ### Cache
	同じ URL の繰り返し取得を減らすため、結果は 15 分間（設定可能）キャッシュされます。

## 設定

json5

```
{
  tools: {
    web: {
      fetch: {
        enabled: true, // default: true
        provider: "firecrawl", // optional; omit for auto-detect
        maxChars: 50000, // max output chars
        maxCharsCap: 50000, // hard cap for maxChars param
        maxResponseBytes: 2000000, // max download size before truncation
        timeoutSeconds: 30,
        cacheTtlMinutes: 15,
        maxRedirects: 3,
        useTrustedEnvProxy: false, // let a trusted HTTP(S) env proxy resolve DNS
        readability: true, // use Readability extraction
        userAgent: "Mozilla/5.0 ...", // override User-Agent
        ssrfPolicy: {
          allowRfc2544BenchmarkRange: true, // opt-in for trusted fake-IP proxies using 198.18.0.0/15
          allowIpv6UniqueLocalRange: true, // opt-in for trusted fake-IP proxies using fc00::/7
        },
      },
    },
  },
}
```

## Firecrawl フォールバック

Readability 抽出が失敗した場合、 `web_fetch` は bot 回避とより優れた抽出のために [Firecrawl](https://docs.openclaw.ai/ja-JP/tools/firecrawl) へフォールバックできます。

json5

```
{
  tools: {
    web: {
      fetch: {
        provider: "firecrawl", // optional; omit for auto-detect from available credentials
      },
    },
  },
  plugins: {
    entries: {
      firecrawl: {
        enabled: true,
        config: {
          webFetch: {
            apiKey: "fc-...", // optional if FIRECRAWL_API_KEY is set
            baseUrl: "https://api.firecrawl.dev",
            onlyMainContent: true,
            maxAgeMs: 86400000, // cache duration (1 day)
            timeoutSeconds: 60,
          },
        },
      },
    },
  },
}
```

`plugins.entries.firecrawl.config.webFetch.apiKey` は SecretRef オブジェクトをサポートします。 レガシーの `tools.web.fetch.firecrawl.*` 設定は `openclaw doctor --fix` によって自動移行されます。

> [!note] Note
> **Note**
> 
> Firecrawl が有効で、その SecretRef が解決されず、 `FIRECRAWL_API_KEY` env フォールバックもない場合、 gateway の起動は早期に失敗します。

> [!note] Note
> **Note**
> 
> Firecrawl の `baseUrl` オーバーライドは制限されています。ホスト型トラフィックは `https://api.firecrawl.dev` を使用します。セルフホストのオーバーライドはプライベートまたは 内部エンドポイントを対象にする必要があり、 `http://` はそれらのプライベート対象にのみ受け入れられます。

現在のランタイム動作:

- `tools.web.fetch.provider` は取得フォールバックプロバイダーを明示的に選択します。
- `provider` が省略された場合、OpenClaw は利用可能な認証情報から、準備済みの最初の web-fetch プロバイダーを自動検出します。サンドボックス外の `web_fetch` は、 `contracts.webFetchProviders` を宣言し、実行時に一致するプロバイダーを登録する インストール済み plugins を使用できます。現在、バンドルされているプロバイダーは Firecrawl です。
- サンドボックス化された `web_fetch` 呼び出しは、バンドルされたプロバイダーに限定されたままです。
- Readability が無効な場合、 `web_fetch` は選択されたプロバイダーフォールバックへ直接進みます。 利用可能なプロバイダーがない場合は、クローズドに失敗します。

## 信頼済み env プロキシ

デプロイで `web_fetch` が信頼済みのアウトバウンド HTTP(S) プロキシを通る必要がある場合は、 `tools.web.fetch.useTrustedEnvProxy: true` を設定します。

このモードでも、OpenClaw はリクエスト送信前にホスト名ベースの SSRF チェックを適用しますが、 ローカル DNS ピンニングを行う代わりに、プロキシに DNS 解決を任せます。 これは、プロキシが運用者によって管理され、DNS 解決後にアウトバウンドポリシーを強制する場合にのみ有効にしてください。

> [!note] Note
> **Note**
> 
> HTTP(S) プロキシ env var が設定されていない場合、または対象ホストが `NO_PROXY` によって除外されている場合、 `web_fetch` は local DNS ピンニングを使用する通常の厳格なパスにフォールバックします。

## 制限と安全性

- `maxChars` は `tools.web.fetch.maxCharsCap` にクランプされます
- レスポンス本文は解析前に `maxResponseBytes` で上限設定されます。過大な レスポンスは警告付きで切り詰められます
- プライベート/内部ホスト名はブロックされます
- `tools.web.fetch.ssrfPolicy.allowRfc2544BenchmarkRange` と `tools.web.fetch.ssrfPolicy.allowIpv6UniqueLocalRange` は、信頼済みの fake-IP プロキシスタック向けの狭いオプトインです。 それらの合成範囲をプロキシが所有し、独自の宛先ポリシーを強制している場合を除き、未設定のままにしてください
- リダイレクトはチェックされ、 `maxRedirects` によって制限されます
- `useTrustedEnvProxy` は明示的なオプトインであり、DNS 解決後もアウトバウンドポリシーを強制する 運用者管理のプロキシに対してのみ有効にしてください
- `web_fetch` はベストエフォートです -- 一部のサイトには [Webブラウザー](https://docs.openclaw.ai/ja-JP/tools/browser) が必要です

## ツールプロファイル

ツールプロファイルまたは許可リストを使用する場合は、 `web_fetch` または `group:web` を追加します。

json5

```
{
  tools: {
    allow: ["web_fetch"],
    // or: allow: ["group:web"]  (includes web_fetch, web_search, and x_search)
  },
}
```

## 関連

- [Web検索](https://docs.openclaw.ai/ja-JP/tools/web) -- 複数のプロバイダーで Web を検索
- [Webブラウザー](https://docs.openclaw.ai/ja-JP/tools/browser) -- JS が多用されているサイト向けの完全なブラウザー自動化
- [Firecrawl](https://docs.openclaw.ai/ja-JP/tools/firecrawl) -- Firecrawl の検索およびスクレイピングツール