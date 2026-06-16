---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/ollama
source_path: raw/docs/providers/ollama.md
doc_section: providers
title: "Ollama"
ingested: 2026-06-14
tags: [provider, ollama, local-models, self-hosted, cloud, embeddings]
related:
  - "[[providers/ollama]]"
  - "[[concepts/local-models]]"
  - "[[concepts/model-providers]]"
---

# Ollama（解説）

> 原典: `raw/docs/providers/ollama.md` ・ https://docs.openclaw.ai/ja-JP/providers/ollama

## 一言まとめ

Ollama のネイティブ API（`/api/chat`）と統合し、**ローカル/セルフホストとクラウドの Ollama** を扱う。`Cloud + Local`・`Cloud only`（`ollama.com`）・`Local only` の 3 モード。

## 位置づけ

[[providers/ollama]] の詳細ソース。[[concepts/local-models]] の代表的バックエンドで、[[concepts/model-providers]] のカスタム/ローカルプロバイダー。Web 検索（[[sources/tools/ollama-search]]）やメモリ埋め込み（[[sources/plugins/memory-lancedb]]）でも Ollama を使える。

## 仕組み・ふるまい

- **3 モード**：`Cloud + Local`（到達可能ホスト経由）・`Cloud only`（`https://ollama.com`）・`Local only`（到達可能ホスト）。ネイティブ API `/api/chat`。
- 自前ホストの LLM を `provider/model`（`ollama/<model>`）として使える。

## 設定・使い方の要点

- ローカルホストにサインイン（`ollama signin`）、または `OLLAMA_API_KEY` で `ollama.com`。`models.providers.ollama`（baseUrl 等）。`models.mode: "merge"` でホスト型と併存（[[concepts/local-models]]）。

## 注意点・落とし穴

- WSL2＋Ollama＋NVIDIA/CUDA は systemd 自動起動でメモリ固定→VM 再起動ループの注意（[[sources/gateway/local-models]]）。プロバイダー側の安全フィルターは通らない。

## 用語と略称

- **Ollama** = ローカル LLM 実行ツール（ネイティブ API＋クラウド）
- **`/api/chat`** = Ollama のネイティブチャット API
- **Cloud / Local モード** = ホスト型 / 自前サーバーの使い分け
- **埋め込み（embeddings）** = メモリ検索用のベクトル化

## 関連ページ

- [[providers/ollama]] — 対応するプロバイダーページ
- [[concepts/local-models]] / [[concepts/model-providers]] / [[sources/tools/ollama-search]] / [[sources/plugins/memory-lancedb]]
