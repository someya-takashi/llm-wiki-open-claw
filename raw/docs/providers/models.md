---
title: "モデルプロバイダーのクイックスタート"
source: "https://docs.openclaw.ai/ja-JP/providers/models"
author:
published:
created: 2026-06-14
description: "OpenClaw でサポートされているモデルプロバイダー（LLM）"
tags:
  - "clippings"
---
OpenClaw は多くの LLM プロバイダーを使用できます。1 つ選び、認証してから、既定の モデルを `provider/model` として設定します。

## クイックスタート（2 ステップ）

1. プロバイダーで認証します（通常は `openclaw onboard` 経由）。
2. 既定のモデルを設定します。

json5

```
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## サポートされるプロバイダー（スターターセット）

- [Alibaba Model Studio](https://docs.openclaw.ai/ja-JP/providers/alibaba)
- [Amazon Bedrock](https://docs.openclaw.ai/ja-JP/providers/bedrock)
- [Anthropic (API + Claude CLI)](https://docs.openclaw.ai/ja-JP/providers/anthropic)
- [BytePlus (International)](https://docs.openclaw.ai/ja-JP/concepts/model-providers#byteplus-international)
- [Chutes](https://docs.openclaw.ai/ja-JP/providers/chutes)
- [ComfyUI](https://docs.openclaw.ai/ja-JP/providers/comfy)
- [Cloudflare AI Gateway](https://docs.openclaw.ai/ja-JP/providers/cloudflare-ai-gateway)
- [DeepInfra](https://docs.openclaw.ai/ja-JP/providers/deepinfra)
- [fal](https://docs.openclaw.ai/ja-JP/providers/fal)
- [Fireworks](https://docs.openclaw.ai/ja-JP/providers/fireworks)
- [GLM モデル](https://docs.openclaw.ai/ja-JP/providers/glm)
- [MiniMax](https://docs.openclaw.ai/ja-JP/providers/minimax)
- [Mistral](https://docs.openclaw.ai/ja-JP/providers/mistral)
- [Moonshot AI (Kimi + Kimi Coding)](https://docs.openclaw.ai/ja-JP/providers/moonshot)
- [OpenAI (API + Codex)](https://docs.openclaw.ai/ja-JP/providers/openai)
- [OpenCode (Zen + Go)](https://docs.openclaw.ai/ja-JP/providers/opencode)
- [OpenRouter](https://docs.openclaw.ai/ja-JP/providers/openrouter)
- [Qianfan](https://docs.openclaw.ai/ja-JP/providers/qianfan)
- [Qwen](https://docs.openclaw.ai/ja-JP/providers/qwen)
- [Runway](https://docs.openclaw.ai/ja-JP/providers/runway)
- [StepFun](https://docs.openclaw.ai/ja-JP/providers/stepfun)
- [Synthetic](https://docs.openclaw.ai/ja-JP/providers/synthetic)
- [Vercel AI Gateway](https://docs.openclaw.ai/ja-JP/providers/vercel-ai-gateway)
- [Venice (Venice AI)](https://docs.openclaw.ai/ja-JP/providers/venice)
- [xAI](https://docs.openclaw.ai/ja-JP/providers/xai)
- [Z.AI](https://docs.openclaw.ai/ja-JP/providers/zai)

## 追加の同梱プロバイダーバリアント

- `anthropic-vertex` - Vertex 認証情報が利用可能な場合の、Google Vertex 上の Anthropic の暗黙的なサポート。別個のオンボーディング認証選択肢はありません
- `copilot-proxy` - ローカルの VS Code Copilot Proxy ブリッジ。 `openclaw onboard --auth-choice copilot-proxy` を使用します
- `google-gemini-cli` - 非公式の Gemini CLI OAuth フロー。ローカルの `gemini` インストールが必要です（ `brew install gemini-cli` または `npm install -g @google/gemini-cli` ）。既定のモデルは `google-gemini-cli/gemini-3-flash-preview` です。 `openclaw onboard --auth-choice google-gemini-cli` または `openclaw models auth login --provider google-gemini-cli --set-default` を使用します

完全なプロバイダーカタログ（xAI、Groq、Mistral など）と高度な設定については、 [モデルプロバイダー](https://docs.openclaw.ai/ja-JP/concepts/model-providers) を参照してください。

## 関連

- [モデル選択](https://docs.openclaw.ai/ja-JP/concepts/model-providers)
- [モデルフェイルオーバー](https://docs.openclaw.ai/ja-JP/concepts/model-failover)
- [Models CLI](https://docs.openclaw.ai/ja-JP/cli/models)