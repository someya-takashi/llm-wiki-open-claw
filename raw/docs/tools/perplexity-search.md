---
title: "Perplexity 検索"
source: "https://docs.openclaw.ai/ja-JP/tools/perplexity-search"
author:
published:
created: 2026-06-14
description: "web_search 向け Perplexity Search API と Sonar/OpenRouter の互換性"
tags:
  - "clippings"
---
OpenClaw は Perplexity Search API を `web_search` プロバイダーとしてサポートします。 これは `title` 、 `url` 、 `snippet` フィールドを持つ構造化された結果を返します。

互換性のため、OpenClaw は従来の Perplexity Sonar/OpenRouter セットアップもサポートします。 `OPENROUTER_API_KEY` 、 `plugins.entries.perplexity.config.webSearch.apiKey` 内の `sk-or-...` キーを使用するか、 `plugins.entries.perplexity.config.webSearch.baseUrl` / `model` を設定すると、プロバイダーは chat-completions パスに切り替わり、構造化された Search API 結果ではなく、引用付きの AI 合成回答を返します。

## Perplexity API キーの取得

1. [perplexity.ai/settings/api](https://www.perplexity.ai/settings/api) で Perplexity アカウントを作成する
2. ダッシュボードで API キーを生成する
3. キーを設定に保存するか、Gateway 環境で `PERPLEXITY_API_KEY` を設定します。

## OpenRouter 互換性

すでに Perplexity Sonar 用に OpenRouter を使用していた場合は、 `provider: "perplexity"` を維持し、Gateway 環境で `OPENROUTER_API_KEY` を設定するか、 `plugins.entries.perplexity.config.webSearch.apiKey` に `sk-or-...` キーを保存します。

任意の互換性制御:

- `plugins.entries.perplexity.config.webSearch.baseUrl`
- `plugins.entries.perplexity.config.webSearch.model`

## 設定例

### ネイティブ Perplexity Search API

json5

```
{
  plugins: {
    entries: {
      perplexity: {
        config: {
          webSearch: {
            apiKey: "pplx-...",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "perplexity",
      },
    },
  },
}
```

### OpenRouter / Sonar 互換性

json5

```
{
  plugins: {
    entries: {
      perplexity: {
        config: {
          webSearch: {
            apiKey: "<openrouter-api-key>",
            baseUrl: "https://openrouter.ai/api/v1",
            model: "perplexity/sonar-pro",
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "perplexity",
      },
    },
  },
}
```

## キーの設定場所

**設定経由:** `openclaw configure --section web` を実行します。これはキーを `plugins.entries.perplexity.config.webSearch.apiKey` の下の `~/.openclaw/openclaw.json` に保存します。 このフィールドは SecretRef オブジェクトも受け入れます。

**環境経由:** Gateway プロセス環境で `PERPLEXITY_API_KEY` または `OPENROUTER_API_KEY` を設定します。Gateway インストールの場合は、 `~/.openclaw/.env` （またはサービス環境）に配置します。 [環境変数](https://docs.openclaw.ai/ja-JP/help/faq#env-vars-and-env-loading) を参照してください。

`provider: "perplexity"` が設定されていて、Perplexity キー SecretRef が未解決で env フォールバックもない場合、起動/再読み込みは即座に失敗します。

## ツールパラメーター

これらのパラメーターは、ネイティブ Perplexity Search API パスに適用されます。

検索クエリ。

返す結果数（1〜10）。

2 文字の ISO 国コード（例: `US` 、 `DE` ）。

ISO 639-1 言語コード（例: `en` 、 `de` 、 `fr` ）。

時間フィルター - `day` は 24 時間です。

この日付（ `YYYY-MM-DD` ）より後に公開された結果のみ。

この日付（ `YYYY-MM-DD` ）より前に公開された結果のみ。

ドメインの許可リスト/拒否リスト配列（最大 20）。

合計コンテンツ予算（最大 1000000）。

ページごとのトークン上限。

従来の Sonar/OpenRouter 互換性パスの場合:

- `query` 、 `count` 、 `freshness` が受け入れられます
- そこでの `count` は互換性専用です。レスポンスは N 件の結果リストではなく、引用付きの 1 つの合成回答のままです
- `country` 、 `language` 、 `date_after` 、 `date_before` 、 `domain_filter` 、 `max_tokens` 、 `max_tokens_per_page` などの Search API 専用フィルターは明示的なエラーを返します

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
 
// Domain filtering (allowlist)
await web_search({
  query: "climate research",
  domain_filter: ["nature.com", "science.org", ".edu"],
});
 
// Domain filtering (denylist - prefix with -)
await web_search({
  query: "product reviews",
  domain_filter: ["-reddit.com", "-pinterest.com"],
});
 
// More content extraction
await web_search({
  query: "detailed AI research",
  max_tokens: 50000,
  max_tokens_per_page: 4096,
});
```

### ドメインフィルターのルール

- フィルターごとに最大 20 ドメイン
- 同じリクエスト内で許可リストと拒否リストを混在させることはできません
- 拒否リスト項目には `-` プレフィックスを使用します（例: `["-reddit.com"]` ）

## 注記

- Perplexity Search API は構造化された Web 検索結果（ `title` 、 `url` 、 `snippet` ）を返します
- OpenRouter または明示的な `plugins.entries.perplexity.config.webSearch.baseUrl` / `model` は、互換性のために Perplexity を Sonar chat completions に戻します
- Sonar/OpenRouter 互換性は、構造化された結果行ではなく、引用付きの 1 つの合成回答を返します
- 結果はデフォルトで 15 分間キャッシュされます（ `cacheTtlMinutes` で設定可能）

## 関連[**Web search overview**

すべてのプロバイダーと自動検出ルール。

](https://docs.openclaw.ai/ja-JP/tools/web)

[

**Brave search**

国と言語フィルター付きの構造化された結果。

](https://docs.openclaw.ai/ja-JP/tools/brave-search)[

**Exa search**

コンテンツ抽出付きのニューラル検索。

](https://docs.openclaw.ai/ja-JP/tools/exa-search)[

**Perplexity Search API docs**

公式 Perplexity Search API クイックスタートとリファレンス。

](https://docs.perplexity.ai/docs/search/quickstart)