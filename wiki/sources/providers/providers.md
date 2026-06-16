---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers
source_path: raw/docs/providers/providers.md
doc_section: providers
title: "プロバイダーディレクトリ"
ingested: 2026-06-14
tags: [providers, directory, llm, catalog, model-reference]
related:
  - "[[concepts/model-providers]]"
  - "[[components/gateway]]"
---

# プロバイダーディレクトリ（解説）

> 原典: `raw/docs/providers/providers.md` ・ https://docs.openclaw.ai/ja-JP/providers
>
> ℹ️ `providers/` セクションのランディング。50 以上の LLM プロバイダーへのカタログ。

## 一言まとめ

OpenClaw が使える**全 LLM プロバイダーのディレクトリ**。プロバイダーを選んで認証し、既定モデルを `provider/model` として設定する。

## 位置づけ

[[concepts/model-providers]] のカタログソース。チャットチャネル（[[concepts/channel-routing]]）とは別物。

## 仕組み・ふるまい（主要プロバイダー）

- **主要 API**：[[providers/anthropic]]（Claude）・[[providers/openai]]（GPT/Codex）・[[providers/google]]（Gemini）・OpenRouter・Groq・Mistral・xAI・DeepSeek・MiniMax・Moonshot。
- **クラウド/エンタープライズ**：[[providers/bedrock]]（AWS）・Vertex・Azure・GitHub Copilot・Cloudflare AI Gateway・[[providers/litellm]]（統合 Gateway）・Vercel AI Gateway。
- **ローカル/セルフホスト**：[[providers/ollama]]・LM Studio・inferrs（[[concepts/local-models]]）。
- **メディア専用**：ElevenLabs・fal・ComfyUI・Runway 等（音声/画像/動画生成）。

## 設定・使い方の要点

- クイックスタート：`openclaw onboard` で認証 → `agents.defaults.model.primary` に `provider/model`。詳細は各 `providers/<slug>` ページ。

## 用語と略称

- **`provider/model`** = モデル参照の形式
- **統合 Gateway** = 複数プロバイダーを 1 API に束ねる中継（LiteLLM 等）
- **AI Gateway** = ルーティング/キャッシュ/監視を挟む中継層

## 関連ページ

- [[concepts/model-providers]] — 対応する概念ページ
- [[sources/providers/models]] / [[providers/anthropic]] / [[providers/openai]] / [[providers/google]] / [[providers/ollama]]
