---
title: "Brave 検索"
source: "https://docs.openclaw.ai/ja-JP/tools/brave-search"
author:
published:
created: 2026-06-14
description: "web_search 用の Brave Search API セットアップ"
tags:
  - "clippings"
---
OpenClaw は Brave Search API を `web_search` プロバイダーとしてサポートしています。

## API キーを取得する

1. [https://brave.com/search/api/](https://brave.com/search/api/) で Brave Search API アカウントを作成します。
2. ダッシュボードで **Search** プランを選択し、API キーを生成します。
3. キーを config に保存するか、Gateway 環境で `BRAVE_API_KEY` を設定します。

## Config の例

json5

```
{
  plugins: {
    entries: {
      brave: {
        config: {
          webSearch: {
            apiKey: "BRAVE_API_KEY_HERE",
            mode: "web", // or "llm-context"
            baseUrl: "https://api.search.brave.com", // optional proxy/base URL override
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "brave",
        maxResults: 5,
        timeoutSeconds: 30,
      },
    },
  },
}
```

プロバイダー固有の Brave 検索設定は、現在 `plugins.entries.brave.config.webSearch.*` 配下にあります。 従来の `tools.web.search.apiKey` は互換性 shim を通じて引き続き読み込まれますが、標準の config パスではなくなりました。

`webSearch.mode` は Brave のトランスポートを制御します。

- `web` （デフォルト）: タイトル、URL、スニペットを含む通常の Brave Web 検索
- `llm-context`: grounding 用に事前抽出されたテキストチャンクとソースを含む Brave LLM Context API

`webSearch.baseUrl` は、Brave リクエストを信頼済みの Brave 互換プロキシ または Gateway に向けることができます。OpenClaw は設定されたベース URL に `/res/v1/web/search` または `/res/v1/llm/context` を追加し、ベース URL をキャッシュキーに保持します。公開 エンドポイントでは `https://` を使用する必要があります。 `http://` は、信頼済みの loopback またはプライベートネットワークのプロキシホストに限り受け入れられます。

## ツールパラメーター

検索クエリ。

返す結果数（1–10）。

2 文字の ISO 国コード（例: `US`, `DE` ）。

検索結果用の ISO 639-1 言語コード（例: `en`, `de`, `fr` ）。

Brave 検索言語コード（例: `en`, `en-gb`, `zh-hans` ）。

UI 要素用の ISO 言語コード。

時間フィルター — `day` は 24 時間です。

この日付（ `YYYY-MM-DD` ）より後に公開された結果のみ。

この日付（ `YYYY-MM-DD` ）より前に公開された結果のみ。

**例:**

javascript

```javascript
// Country and language-specific search
await web_search({
  query: "renewable energy",
  country: "DE",
  language: "de",
});
 
// Recent results (past week)
await web_search({
  query: "AI news",
  freshness: "week",
});
 
// Date range search
await web_search({
  query: "AI developments",
  date_after: "2024-01-01",
  date_before: "2024-06-30",
});
```

## 注記

- OpenClaw は Brave **Search** プランを使用します。従来のサブスクリプション（例: 月 2,000 クエリの元の Free プラン）がある場合、それは引き続き有効ですが、LLM Context やより高いレート制限などの新しい機能は含まれません。
- 各 Brave プランには、 **月額 $5 の無料クレジット** （更新あり）が含まれます。Search プランの料金は 1,000 リクエストあたり $5 なので、このクレジットで月 1,000 クエリをカバーできます。予期しない課金を避けるため、Brave ダッシュボードで使用上限を設定してください。現在のプランについては [Brave API ポータル](https://brave.com/search/api/) を参照してください。
- Search プランには LLM Context エンドポイントと AI 推論権が含まれます。モデルのトレーニングやチューニングのために結果を保存するには、明示的な保存権を含むプランが必要です。Brave の [Terms of Service](https://api-dashboard.search.brave.com/terms-of-service) を参照してください。
- `llm-context` モードは、通常の Web 検索スニペット形式の代わりに、grounding されたソースエントリを返します。
- `llm-context` モードは `freshness` と、範囲が指定された `date_after` + `date_before` をサポートします。 `ui_lang` はサポートしていません。Brave ではカスタム freshness 範囲に開始日と終了日の両方が必要なため、 `date_after` なしの `date_before` は拒否されます。
- `ui_lang` には `en-US` のような地域サブタグを含める必要があります。
- 結果はデフォルトで 15 分間キャッシュされます（ `cacheTtlMinutes` で設定可能）。
- カスタム `webSearch.baseUrl` 値は Brave キャッシュ ID に含まれるため、 プロキシ固有のレスポンスが衝突しません。
- トラブルシューティング中に Brave リクエスト URL/クエリパラメーター、レスポンスステータス/タイミング、検索キャッシュのヒット/ミス/書き込みイベントをログに記録するには、 `brave.http` 診断フラグを有効にします。このフラグは API キーやレスポンス本文をログに記録することはありませんが、検索クエリは機密性が高い場合があります。

## 関連

- [Web Search の概要](https://docs.openclaw.ai/ja-JP/tools/web) -- すべてのプロバイダーと自動検出
- [Perplexity Search](https://docs.openclaw.ai/ja-JP/tools/perplexity-search) -- ドメインフィルタリング付きの構造化された結果
- [Exa Search](https://docs.openclaw.ai/ja-JP/tools/exa-search) -- コンテンツ抽出付きのニューラル検索