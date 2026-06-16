---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/litellm
source_path: raw/docs/providers/litellm.md
doc_section: providers
title: "LiteLLM"
ingested: 2026-06-14
tags: [provider, litellm, gateway, unified-api, cost-tracking, proxy]
related:
  - "[[providers/litellm]]"
  - "[[concepts/model-providers]]"
---

# LiteLLM（解説）

> 原典: `raw/docs/providers/litellm.md` ・ https://docs.openclaw.ai/ja-JP/providers/litellm

## 一言まとめ

LiteLLM は **100 以上のプロバイダーに統一 API を提供する OSS の LLM Gateway**。OpenClaw を LiteLLM 経由でルーティングすると、コスト追跡・ロギング・OpenClaw 設定を変えずにバックエンド切替を一元化できる。

## 位置づけ

[[providers/litellm]] の詳細ソース。[[concepts/model-providers]] の「統合 Gateway」系（Vercel/Cloudflare AI Gateway と同種）。OpenClaw から見れば OpenAI 互換のカスタムプロバイダー。

## 仕組み・ふるまい

- LiteLLM サーバーを立て、OpenAI 互換エンドポイントとして `models.providers.<id>`（baseUrl＝LiteLLM URL）に登録。OpenClaw は 1 つのプロバイダーとして扱い、実バックエンドは LiteLLM が振り分ける。

## 設定・使い方の要点

- `models.providers.litellm`（`baseUrl`・`apiKey`・`api: "openai-completions"`・`models[]`）。LiteLLM 側でルーティング/コスト/ロギングを設定。

## 注意点・落とし穴

- OpenClaw のモデルフェイルオーバー（[[concepts/model-failover]]）と LiteLLM のルーティングは別レイヤー——二重化すると挙動が読みづらくなるので役割を分ける。

## 用語と略称

- **LiteLLM** = 統一 API を提供する OSS の LLM Gateway
- **統合 Gateway** = 複数プロバイダーを 1 API に束ねる中継
- **OpenAI 互換** = `/v1/chat/completions` 形式の API

## 関連ページ

- [[providers/litellm]] — 対応するプロバイダーページ
- [[concepts/model-providers]] / [[concepts/local-models]] / [[concepts/model-failover]]
