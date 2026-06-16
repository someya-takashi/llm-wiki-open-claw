---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/local-models
source_path: raw/docs/gateway/local-models.md
doc_section: gateway
title: "ローカルモデル"
ingested: 2026-06-14
tags: [local-models, lm-studio, ollama, openai-compat, privacy, self-hosted]
related:
  - "[[concepts/local-models]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/security]]"
---

# ローカルモデル（解説）

> 原典: `raw/docs/gateway/local-models.md` ・ https://docs.openclaw.ai/ja-JP/gateway/local-models

## 一言まとめ

OpenClaw を**自前ホストのローカル LLM**で動かすための意見つきガイド。実現可能だが、ハードウェア・コンテキスト長・プロンプトインジェクション防御の要求水準が上がる。「常に**ホストできる最大/フルサイズ**を使え、小型・過量子化は避けろ」が一貫した主張。

## 位置づけ

[[concepts/local-models]] の本体。[[concepts/agent-runtimes]] の provider/model 層に OpenAI 互換のローカルサーバーを登録する話で、必要時だけ起動するなら [[sources/gateway/local-model-services]]、API 障害時のテキスト専用フォールバックは [[sources/gateway/cli-backends]]。

## 仕組み・ふるまい

- **バックエンド選択**：LM Studio（GUI・ネイティブ Responses API、初回向け）／Ollama（CLI・モデルライブラリ・systemd）／MLX・vLLM・SGLang（高スループット OpenAI 互換 HTTP）／LiteLLM・カスタムプロキシ。対応すれば `api: "openai-responses"`、なければ `api: "openai-completions"`。
- **推奨スタック**：LM Studio ＋大規模モデル＋ Responses API。`models.providers.<id>` に `baseUrl`（既定 `http://127.0.0.1:1234/v1`）・`apiKey`・`api`・`models[]`（`id`/`contextWindow`/`maxTokens` 等）を定義。
- **ハイブリッド/フォールバック**：`models.mode: "merge"` でホスト型モデルを残し、`model.primary`/`fallbacks` でローカル⇄ホストの安全網を構成（[[concepts/retry]] / モデルフェイルオーバー）。

## 設定・使い方の要点

- **ハードウェア下限**：快適なエージェントループには最大構成 Mac Studio ×2 以上 or 同等 GPU リグ（~$30k+）が目安。単一 24GB GPU は軽量・高レイテンシ用。
- ループバック `127.0.0.1` は自動信頼。LAN/tailnet/`.local` エンドポイントは `request.allowPrivateNetwork: true` が必要。ローカルマーカー（`apiKey: "ollama-local"` 等）はループバック/プライベート解決時のみ許容。
- **互換シム**：`compat.requiresStringContent`（content 文字列のみ受理するサーバー）、`compat.strictMessageKeys`、`compat.supportsTools: false`、`params.extra_body.tool_choice: "required"`（ツール強制が要るサーバー）。
- **小型/厳格バックエンドの切り分け**：`openclaw infer model run --local`／`--gateway` でトランスポートを段階確認 → `experimental.localModelLean: true`（重い `browser`/`cron`/`message` を外す）→ `compat.supportsTools: false` → それ以降は上流容量の問題。

## 注意点・落とし穴

- ⚠️ **ローカルモデルはプロバイダー側の安全フィルターを通らない**。過量子化・小型はプロンプトインジェクション耐性が落ちる。エージェントを狭く保ち [[concepts/compaction]] を有効に（[[concepts/security]]）。
- ローカル/プロキシ `/v1` は「プロキシ形式の OpenAI 互換」扱い：OpenAI 専用整形（`service_tier`・`store`・プロンプトキャッシュ）や帰属ヘッダーは注入されない。
- ツール呼び出しが生 JSON/XML/ReAct テキストで出る場合、まずサーバーのチャットテンプレート/パーサーを直す（盲目的にテキスト→ツール昇格しない）。

## 用語と略称

- **量子化（quantization）** = 重みを低ビット化してメモリ/速度を稼ぐ圧縮
- **OpenAI 互換 `/v1`** = `/v1/chat/completions`・`/v1/responses` を話す自前サーバー
- **GGUF** = ローカル推論で広く使われるモデル重みフォーマット
- **kv-cache** = 注意機構の中間状態キャッシュ（長文で GPU メモリを消費）
- **MLX / vLLM / SGLang** = 高速ローカル推論サーバー実装

## 関連ページ

- [[concepts/local-models]] — 対応する概念ページ
- [[sources/gateway/local-model-services]] / [[sources/gateway/cli-backends]]
- [[concepts/agent-runtimes]] / [[concepts/retry]] / [[concepts/security]] / [[providers/lmstudio]]（未作成） / [[providers/ollama]]
