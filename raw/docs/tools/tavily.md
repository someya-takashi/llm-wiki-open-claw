---
title: "Tavily"
source: "https://docs.openclaw.ai/ja-JP/tools/tavily"
author:
published:
created: 2026-06-14
description: "Tavily の検索および抽出ツール"
tags:
  - "clippings"
---
[Tavily](https://tavily.com/) は、AIアプリケーション向けに設計された検索APIです。OpenClaw では、次の2つの方法で公開されています。

- 汎用検索ツールの `web_search` provider として
- 明示的な Plugin ツールとして: `tavily_search` と `tavily_extract`

Tavily は、設定可能な検索深度、トピックフィルタリング、ドメインフィルター、AI生成の回答要約、URLからのコンテンツ抽出（JavaScriptでレンダリングされるページを含む）を備えた、LLMでの利用に最適化された構造化結果を返します。

| プロパティ | 値 |
| --- | --- |
| Plugin ID | `tavily` |
| 認証 | `TAVILY_API_KEY` または config `apiKey` |
| ベースURL | `https://api.tavily.com` (デフォルト) |
| 同梱ツール | `tavily_search`, `tavily_extract` |

## はじめに

- ### APIキーを取得する
	[tavily.com](https://tavily.com/) で Tavily アカウントを作成し、ダッシュボードでAPIキーを生成します。
- ### Plugin と provider を設定する
	json5
	```
	{
	  plugins: {
	    entries: {
	      tavily: {
	        enabled: true,
	        config: {
	          webSearch: {
	            apiKey: "tvly-...", // optional if TAVILY_API_KEY is set
	            baseUrl: "https://api.tavily.com",
	          },
	        },
	      },
	    },
	  },
	  tools: {
	    web: {
	      search: {
	        provider: "tavily",
	      },
	    },
	  },
	}
	```
- ### 検索が実行されることを確認する
	任意の agent から `web_search` をトリガーするか、 `tavily_search` を直接呼び出します。

> [!note] Note
> **Tip**
> 
> オンボーディングまたは `openclaw configure --section web` で Tavily を選択すると、同梱の Tavily Plugin が自動的に有効になります。

## ツールリファレンス

### tavily\_search

汎用の `web_search` ではなく、Tavily 固有の検索制御を使用したい場合に使います。

| パラメーター | 型 | 制約 / デフォルト | 説明 |
| --- | --- | --- | --- |
| `query` | 文字列 | 必須 | 検索クエリ文字列。400文字未満にしてください。 |
| `search_depth` | 列挙型 | `basic` (デフォルト), `advanced` | `advanced` は遅いものの、関連性が高くなります。 |
| `topic` | 列挙型 | `general` (デフォルト), `news`, `finance` | トピックファミリーでフィルタリングします。 |
| `max_results` | 整数 | 1-20 | 結果の数。 |
| `include_answer` | 真偽値 | デフォルト `false` | Tavily のAI生成回答要約を含めます。 |
| `time_range` | 列挙型 | `day`, `week`, `month`, `year` | 新しさで結果をフィルタリングします。 |
| `include_domains` | 文字列配列 | (なし) | これらのドメインからの結果のみを含めます。 |
| `exclude_domains` | 文字列配列 | (なし) | これらのドメインからの結果を除外します。 |

検索深度のトレードオフ:

| 深度 | 速度 | 関連性 | 最適な用途 |
| --- | --- | --- | --- |
| `basic` | 高速 | 高い | 汎用クエリ (デフォルト)。 |
| `advanced` | 低速 | 最高 | 精密な調査と事実確認。 |

### tavily\_extract

1つ以上のURLからクリーンなコンテンツを抽出するために使います。JavaScriptでレンダリングされるページに対応し、対象を絞った抽出のためにクエリ重視のチャンク化をサポートします。

| パラメーター | 型 | 制約 / デフォルト | 説明 |
| --- | --- | --- | --- |
| `urls` | 文字列配列 | 必須、1-20 | コンテンツ抽出元のURL。 |
| `query` | 文字列 | (任意) | 抽出されたチャンクをこのクエリとの関連性で再ランク付けします。 |
| `extract_depth` | 列挙型 | `basic` (デフォルト), `advanced` | JSの多いページ、SPA、動的テーブルには `advanced` を使います。 |
| `chunks_per_source` | 整数 | 1-5; **`query` が必要** | URLごとに返されるチャンク数。 `query` なしで設定するとエラーになります。 |
| `include_images` | 真偽値 | デフォルト `false` | 結果に画像URLを含めます。 |

抽出深度のトレードオフ:

| 深度 | 使用する場面 |
| --- | --- |
| `basic` | 単純なページ。まずはこちらを試してください。 |
| `advanced` | JSでレンダリングされるSPA、動的コンテンツ、テーブル。 |

> [!note] Note
> **Tip**
> 
> 大きなURLリストは複数の `tavily_extract` 呼び出しに分割してください（1リクエストあたり最大20件）。ページ全体ではなく関連コンテンツのみを取得するには、 `query` と `chunks_per_source` を併用します。

## 適切なツールの選択

| 必要なこと | ツール |
| --- | --- |
| 特別なオプションなしのクイックWeb検索 | `web_search` |
| 深度、トピック、AI回答を指定した検索 | `tavily_search` |
| 特定URLからのコンテンツ抽出 | `tavily_extract` |

> [!note] Note
> **Note**
> 
> Tavily を provider とする汎用 `web_search` ツールは、 `query` と `count` （最大20件の結果）をサポートします。Tavily 固有の制御（ `search_depth` 、 `topic` 、 `include_answer` 、ドメインフィルター、時間範囲）には、代わりに `tavily_search` を使います。

## 高度な設定

APIキーの解決順序

Tavily クライアントは、次の順序でAPIキーを検索します。

どちらも存在しない場合、 `tavily_extract` はセットアップエラーを発生させます。

カスタムベースURL

プロキシ経由で Tavily を利用する場合は、 `plugins.entries.tavily.config.webSearch.baseUrl` を上書きします。デフォルトは `https://api.tavily.com` です。

\`chunks\_per\_source\` には \`query\` が必要

`tavily_extract` は、 `query` なしで `chunks_per_source` を渡す呼び出しを拒否します。Tavily はクエリとの関連性でチャンクをランク付けするため、このパラメーターはクエリなしでは意味を持ちません。

## 関連[**Web Search の概要**

すべての provider と自動検出ルール。

](https://docs.openclaw.ai/ja-JP/tools/web)

[

**Firecrawl**

検索に加えてコンテンツ抽出付きのスクレイピング。

](https://docs.openclaw.ai/ja-JP/tools/firecrawl)[

**Exa Search**

コンテンツ抽出付きのニューラル検索。

](https://docs.openclaw.ai/ja-JP/tools/exa-search)[

**設定**

Plugin エントリとツールルーティングの完全な config スキーマ。

](https://docs.openclaw.ai/ja-JP/gateway/configuration)