---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/sdk-provider-plugins
source_path: raw/docs/plugins/sdk-provider-plugins.md
doc_section: plugins
title: "プロバイダー Plugin の構築"
ingested: 2026-06-14
tags: [plugin, sdk, model-provider, llm, catalog, api-key]
related:
  - "[[components/plugin-system]]"
  - "[[sources/plugins/building-plugins]]"
  - "[[concepts/agent-runtimes]]"
---

# プロバイダー Plugin の構築（解説）

> 原典: `raw/docs/plugins/sdk-provider-plugins.md` ・ https://docs.openclaw.ai/ja-JP/plugins/sdk-provider-plugins

## 一言まとめ

**モデルプロバイダー（LLM）を OpenClaw に追加する** provider Plugin の作り方。モデルカタログ・API キー認証・動的モデル解決を備えたプロバイダーを `api.registerProvider(...)` で実装する。

## 位置づけ

[[components/plugin-system]] の `registerProvider` を使う開発ガイド（基礎は [[sources/plugins/building-plugins]]）。完成物は [[concepts/agent-runtimes]] の provider/model 層に乗る（多くの組み込みプロバイダー＝Anthropic/OpenAI/Google… も同型の Plugin）。プロバイダー概念の横断ハブ `model-providers` は今後。

## 仕組み・ふるまい

- **チュートリアル**：プロバイダー登録 → モデルカタログ定義（`models[]`：id/contextWindow/maxTokens/cost/input 等）→ API キー認証 → 動的モデル解決（`provider/*` で実行時にカタログを引く）。
- **カタログ順序**：UI/ピッカーに出る順を制御（[[sources/web/control-ui]] のモデルピッカー、[[concepts/slash-commands]] の `/model`）。

## 設定・使い方の要点

- ファイル構造（マニフェスト＋プロバイダー登録モジュール）。ClawHub への公開手順あり（[[components/clawhub]]）。OpenAI 互換のカスタムプロバイダーは設定だけでも足りる（[[sources/gateway/local-models]]・[[sources/gateway/config-tools]]）が、本格的な統合は Plugin にする。

## 注意点・落とし穴

- モデル参照は `provider/model` 形式。Codex のような特殊ランタイムは provider と runtime を分けて考える（[[concepts/agent-runtimes]]）。

## 用語と略称

- **プロバイダー（provider）** = モデル（LLM）の提供元統合
- **`registerProvider`** = プロバイダーを登録する Plugin API
- **モデルカタログ（catalog）** = 提供モデルの一覧メタデータ
- **動的モデル解決** = 実行時にプロバイダーからモデル一覧を引く仕組み

## 関連ページ

- [[components/plugin-system]] / [[sources/plugins/building-plugins]]
- [[concepts/agent-runtimes]] / [[sources/gateway/local-models]] / [[components/clawhub]]
