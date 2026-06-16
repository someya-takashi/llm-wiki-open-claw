---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/models
source_path: raw/docs/providers/models.md
doc_section: providers
title: "モデルプロバイダーのクイックスタート"
ingested: 2026-06-14
tags: [providers, quickstart, model-reference, onboard, starter-set]
related:
  - "[[concepts/model-providers]]"
  - "[[sources/providers/providers]]"
---

# モデルプロバイダーのクイックスタート（解説）

> 原典: `raw/docs/providers/models.md` ・ https://docs.openclaw.ai/ja-JP/providers/models

## 一言まとめ

プロバイダーを 1 つ選び・認証し・既定モデルを `provider/model` で設定する **2 ステップ**のクイックスタートと、スターターセットのプロバイダー一覧。

## 位置づけ

[[concepts/model-providers]] の入口（全カタログは [[sources/providers/providers]]）。

## 仕組み・ふるまい

1. プロバイダーで認証（通常 `openclaw onboard`）→ 2. `agents.defaults.model.primary` を設定（例 `anthropic/claude-opus-4-6`）。

## 設定・使い方の要点

- スターターセット：Anthropic・[[providers/openai]]（API+Codex）・OpenRouter・Mistral・MiniMax・Moonshot・Qwen・xAI・Z.AI・[[providers/bedrock]]・Fireworks・GLM 等。`anthropic-vertex` のような暗黙バリアントもある。

## 用語と略称

- **`provider/model`** = モデル参照の形式
- **`openclaw onboard`** = 対話的な初期設定ウィザード
- **スターターセット** = 最初に試しやすいプロバイダー群

## 関連ページ

- [[concepts/model-providers]] / [[sources/providers/providers]]
- [[providers/anthropic]] / [[providers/openai]]
