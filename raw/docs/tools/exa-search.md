---
title: "Exa 検索"
source: "https://docs.openclaw.ai/ja-JP/tools/exa-search"
author:
published:
created: 2026-06-14
description: "Exa AI検索 -- コンテンツ抽出付きのニューラル検索とキーワード検索"
tags:
  - "clippings"
---
OpenClaw は、 `web_search` プロバイダーとして [Exa AI](https://exa.ai/) をサポートしています。Exa は、組み込みのコンテンツ抽出（ハイライト、テキスト、要約）を備えたニューラル、キーワード、ハイブリッド検索モードを提供します。

## API キーを取得する

- ### アカウントを作成する
	[exa.ai](https://exa.ai/) でサインアップし、ダッシュボードから API キーを生成します。
- ### キーを保存する
	Gateway 環境に `EXA_API_KEY` を設定するか、次の方法で構成します。
	bash
	```bash
	openclaw configure --section web
	```

## 構成

json5

```
{
  plugins: {
    entries: {
      exa: {
        config: {
          webSearch: {
            apiKey: "exa-...", // optional if EXA_API_KEY is set
            baseUrl: "https://api.exa.ai", // optional; OpenClaw appends /search
          },
        },
      },
    },
  },
  tools: {
    web: {
      search: {
        provider: "exa",
      },
    },
  },
}
```

**環境の代替:** Gateway 環境に `EXA_API_KEY` を設定します。 gateway インストールの場合は、 `~/.openclaw/.env` に配置します。

## Base URL の上書き

Exa 検索リクエストを互換プロキシまたは代替 Exa エンドポイント経由にする必要がある場合は、 `plugins.entries.exa.config.webSearch.baseUrl` を設定します。OpenClaw は、ベアホストの先頭に `https://` を付けて正規化し、パスがすでにそこで終わっていない限り `/search` を追加します。解決されたエンドポイントは検索キャッシュキーに含まれるため、異なる Exa エンドポイントの結果は共有されません。

## ツールパラメーター

検索クエリ。

返す結果数（1–100）。

検索モード。

時間フィルター。

この日付（ `YYYY-MM-DD` ）より後の結果。

この日付（ `YYYY-MM-DD` ）より前の結果。

コンテンツ抽出オプション（以下を参照）。

### コンテンツ抽出

Exa は、検索結果とともに抽出済みコンテンツを返せます。有効にするには `contents` オブジェクトを渡します。

javascript

```javascript
await web_search({
  query: "transformer architecture explained",
  type: "neural",
  contents: {
    text: true, // full page text
    highlights: { numSentences: 3 }, // key sentences
    summary: true, // AI summary
  },
});
```

| コンテンツオプション | 型 | 説明 |
| --- | --- | --- |
| `text` | `boolean \| { maxCharacters }` | ページ全文テキストを抽出する |
| `highlights` | `boolean \| { maxCharacters, query, numSentences, highlightsPerUrl }` | 重要な文を抽出する |
| `summary` | `boolean \| { query }` | AI 生成の要約 |

### 検索モード

| モード | 説明 |
| --- | --- |
| `auto` | Exa が最適なモードを選択する（デフォルト） |
| `neural` | セマンティック/意味ベースの検索 |
| `fast` | 高速なキーワード検索 |
| `deep` | 徹底的なディープ検索 |
| `deep-reasoning` | 推論を伴うディープ検索 |
| `instant` | 最速の結果 |

## 注記

- `contents` オプションが指定されていない場合、Exa はデフォルトで `{ highlights: true }` を使用するため、結果には重要な文の抜粋が含まれます
- 利用可能な場合、結果は Exa API レスポンスの `highlightScores` フィールドと `summary` フィールドを保持します
- 結果の説明は、ハイライト、要約、全文テキストの順に、利用可能なものから解決されます
- `freshness` と `date_after` / `date_before` は組み合わせられません。時間フィルターモードはどちらか一方を使用してください
- クエリごとに最大 100 件の結果を返せます（Exa 検索タイプの制限に従います）
- 結果はデフォルトで 15 分間キャッシュされます（ `cacheTtlMinutes` で構成可能）
- Exa は、構造化 JSON レスポンスを持つ公式 API 統合です

## 関連

- [Web Search の概要](https://docs.openclaw.ai/ja-JP/tools/web) -- すべてのプロバイダーと自動検出
- [Brave Search](https://docs.openclaw.ai/ja-JP/tools/brave-search) -- 国/言語フィルター付きの構造化結果
- [Perplexity Search](https://docs.openclaw.ai/ja-JP/tools/perplexity-search) -- ドメインフィルタリング付きの構造化結果