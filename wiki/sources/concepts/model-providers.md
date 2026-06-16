---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/model-providers
source_path: raw/docs/concepts/model-providers.md
doc_section: concepts
title: "モデルプロバイダー（リファレンス）"
ingested: 2026-06-14
tags: [model-providers, provider-reference, pi-ai-catalog, custom-provider, baseurl]
related:
  - "[[concepts/model-providers]]"
  - "[[concepts/local-models]]"
  - "[[concepts/oauth]]"
---

# モデルプロバイダー（リファレンス）（解説）

> 原典: `raw/docs/concepts/model-providers.md` ・ https://docs.openclaw.ai/ja-JP/concepts/model-providers

## 一言まとめ

全モデルプロバイダーの設定リファレンス。モデル参照 `provider/model` の基本ルール、組み込み（pi-ai カタログ）プロバイダー、`models.providers` 経由のカスタム/ベース URL プロバイダーを網羅する。

## 位置づけ

[[concepts/model-providers]] の中核ソース。モデル選択ルールは [[sources/concepts/models]]、ローカルは [[concepts/local-models]]、サブスク認証は [[concepts/oauth]]。

## 仕組み・ふるまい

- **基本ルール**：モデル参照は `provider/model`。Plugin 所有のプロバイダー動作、API キーローテーション。
- **組み込み（pi-ai カタログ）**：OpenAI（＋Codex OAuth）・Anthropic・Google Gemini（API キー）・Google Vertex・Z.AI(GLM)・Vercel AI Gateway・Kilo Gateway ほか同梱プロバイダー Plugin。認証だけで使える。
- **`models.providers` 経由（カスタム）**：`baseUrl`＋`api`（`openai-completions`/`openai-responses` 等）を宣言。Moonshot(Kimi)・Kimi coding・Volcano Engine(Doubao)・BytePlus・Synthetic・MiniMax・LM Studio・Ollama。

## 設定・使い方の要点

- 組み込みは認証（環境変数/OAuth）→ `agents.defaults.model.primary` に `provider/model`。カスタムは `models.providers.<id>`（baseUrl/apiKey/api/models[]）。`models.mode: "merge"` で組み込みと併存。
- BytePlus 国際は `concepts/model-providers#byteplus-international`（ディレクトリからアンカー参照）。

## 注意点・落とし穴

- 巨大なリファレンス——本ページは個別プロバイダーの全設定を転記せず構造を示す。各プロバイダーの詳細は `providers/<slug>` ページ（[[providers/anthropic]] 等）。

## 用語と略称

- **`provider/model`** = モデル参照の形式（プロバイダー ID / モデル ID）
- **pi-ai カタログ** = OpenClaw 組み込みのモデルカタログ
- **`models.providers`** = カスタムプロバイダーの設定ブロック
- **baseUrl / api** = エンドポイント URL / API 形式（OpenAI 互換等）

## 関連ページ

- [[concepts/model-providers]] — 対応する概念ページ
- [[sources/concepts/models]] / [[sources/concepts/model-failover]]
- [[concepts/local-models]] / [[concepts/oauth]] / [[providers/anthropic]] / [[providers/openai]]
