---
title: "DuckDuckGo 検索"
source: "https://docs.openclaw.ai/ja-JP/tools/duckduckgo-search"
author:
published:
created: 2026-06-14
description: "DuckDuckGo ウェブ検索 -- キー不要のフォールバックプロバイダー (実験的、HTMLベース)"
tags:
  - "clippings"
---
OpenClaw は、 **キー不要** の `web_search` プロバイダーとして DuckDuckGo をサポートしています。API キーやアカウントは不要です。

> [!note] Note
> **Warning**
> 
> DuckDuckGo は、公式 API ではなく DuckDuckGo の非 JavaScript 検索ページから結果を取得する、 **実験的な非公式** 統合です。bot チャレンジページや HTML 変更によって、ときどき壊れる可能性があります。

## セットアップ

API キーは不要です - DuckDuckGo をプロバイダーとして設定するだけです。

- ### Configure
	bash
	```bash
	openclaw configure --section web
	# Select "duckduckgo" as the provider
	```

## 設定

json5

```
{
  tools: {
    web: {
      search: {
        provider: "duckduckgo",
      },
    },
  },
}
```

リージョンと SafeSearch の任意の Plugin レベル設定:

json5

```
{
  plugins: {
    entries: {
      duckduckgo: {
        config: {
          webSearch: {
            region: "us-en", // DuckDuckGo region code
            safeSearch: "moderate", // "strict", "moderate", or "off"
          },
        },
      },
    },
  },
}
```

## ツールパラメーター

検索クエリ。

返す結果数 (1-10)。

DuckDuckGo リージョンコード (例: `us-en`, `uk-en`, `de-de`)。

SafeSearch レベル。

リージョンと SafeSearch は Plugin 設定でも指定できます (上記参照) - ツール パラメーターはクエリごとに設定値を上書きします。

## 注記

- **API キー不要** - そのまま動作し、設定は不要
- **実験的** - 公式 API や SDK ではなく、DuckDuckGo の非 JavaScript HTML 検索ページから結果を収集します
- **bot チャレンジのリスク** - 高負荷または自動化された使用では、DuckDuckGo が CAPTCHA を表示したりリクエストをブロックしたりする場合があります
- **HTML 解析** - 結果はページ構造に依存し、予告なく変更される可能性があります
- **自動検出の順序** - DuckDuckGo は最初のキー不要フォールバック (順序 100) です。設定済みのキーを持つ API ベースのプロバイダーが先に実行され、 その後に Ollama Web Search (順序 110)、SearXNG (順序 200) が実行されます
- **未設定の場合、SafeSearch は moderate がデフォルト** です

> [!note] Note
> **Tip**
> 
> 本番環境で使用する場合は、 [Brave Search](https://docs.openclaw.ai/ja-JP/tools/brave-search) (無料枠あり) または別の API ベースのプロバイダーを検討してください。

## 関連

- [Web Search の概要](https://docs.openclaw.ai/ja-JP/tools/web) -- すべてのプロバイダーと自動検出
- [Brave Search](https://docs.openclaw.ai/ja-JP/tools/brave-search) -- 無料枠付きの構造化された結果
- [Exa Search](https://docs.openclaw.ai/ja-JP/tools/exa-search) -- コンテンツ抽出付きのニューラル検索