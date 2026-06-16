---
title: "プロバイダーディレクトリ"
source: "https://docs.openclaw.ai/ja-JP/providers"
author:
published:
created: 2026-06-14
description: "OpenClaw がサポートするモデルプロバイダー（LLM）"
tags:
  - "clippings"
---
OpenClaw は多くの LLM プロバイダーを利用できます。プロバイダーを選択して認証し、 デフォルトモデルを `provider/model` として設定します。

チャットチャンネルのドキュメント（WhatsApp/Telegram/Discord/Slack/Mattermost (Plugin)/など）を探していますか？ [チャンネル](https://docs.openclaw.ai/ja-JP/channels) を参照してください。

## クイックスタート

1. プロバイダーで認証します（通常は `openclaw onboard` 経由）。
2. デフォルトモデルを設定します。

json5

```
{
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

## プロバイダーのドキュメント

- [Alibaba Model Studio](https://docs.openclaw.ai/ja-JP/providers/alibaba)
- [Amazon Bedrock](https://docs.openclaw.ai/ja-JP/providers/bedrock)
- [Amazon Bedrock Mantle](https://docs.openclaw.ai/ja-JP/providers/bedrock-mantle)
- [Anthropic (API + Claude CLI)](https://docs.openclaw.ai/ja-JP/providers/anthropic)
- [Arcee AI (Trinity models)](https://docs.openclaw.ai/ja-JP/providers/arcee)
- [Azure Speech](https://docs.openclaw.ai/ja-JP/providers/azure-speech)
- [BytePlus (International)](https://docs.openclaw.ai/ja-JP/concepts/model-providers#byteplus-international)
- [Cerebras](https://docs.openclaw.ai/ja-JP/providers/cerebras)
- [Chutes](https://docs.openclaw.ai/ja-JP/providers/chutes)
- [Cloudflare AI Gateway](https://docs.openclaw.ai/ja-JP/providers/cloudflare-ai-gateway)
- [ComfyUI](https://docs.openclaw.ai/ja-JP/providers/comfy)
- [DeepSeek](https://docs.openclaw.ai/ja-JP/providers/deepseek)
- [ElevenLabs](https://docs.openclaw.ai/ja-JP/providers/elevenlabs)
- [fal](https://docs.openclaw.ai/ja-JP/providers/fal)
- [Fireworks](https://docs.openclaw.ai/ja-JP/providers/fireworks)
- [GitHub Copilot](https://docs.openclaw.ai/ja-JP/providers/github-copilot)
- [GLM models](https://docs.openclaw.ai/ja-JP/providers/glm)
- [Google (Gemini)](https://docs.openclaw.ai/ja-JP/providers/google)
- [Gradium](https://docs.openclaw.ai/ja-JP/providers/gradium)
- [Groq (LPU inference)](https://docs.openclaw.ai/ja-JP/providers/groq)
- [Hugging Face (Inference)](https://docs.openclaw.ai/ja-JP/providers/huggingface)
- [inferrs (local models)](https://docs.openclaw.ai/ja-JP/providers/inferrs)
- [Kilocode](https://docs.openclaw.ai/ja-JP/providers/kilocode)
- [LiteLLM (統合 Gateway)](https://docs.openclaw.ai/ja-JP/providers/litellm)
- [LM Studio (local models)](https://docs.openclaw.ai/ja-JP/providers/lmstudio)
- [MiniMax](https://docs.openclaw.ai/ja-JP/providers/minimax)
- [Mistral](https://docs.openclaw.ai/ja-JP/providers/mistral)
- [Moonshot AI (Kimi + Kimi Coding)](https://docs.openclaw.ai/ja-JP/providers/moonshot)
- [NVIDIA](https://docs.openclaw.ai/ja-JP/providers/nvidia)
- [Ollama (クラウド + local models)](https://docs.openclaw.ai/ja-JP/providers/ollama)
- [OpenAI (API + Codex)](https://docs.openclaw.ai/ja-JP/providers/openai)
- [OpenCode](https://docs.openclaw.ai/ja-JP/providers/opencode)
- [OpenCode Go](https://docs.openclaw.ai/ja-JP/providers/opencode-go)
- [OpenRouter](https://docs.openclaw.ai/ja-JP/providers/openrouter)
- [Perplexity (Web 検索)](https://docs.openclaw.ai/ja-JP/providers/perplexity-provider)
- [Qianfan](https://docs.openclaw.ai/ja-JP/providers/qianfan)
- [Qwen Cloud](https://docs.openclaw.ai/ja-JP/providers/qwen)
- [Runway](https://docs.openclaw.ai/ja-JP/providers/runway)
- [SenseAudio](https://docs.openclaw.ai/ja-JP/providers/senseaudio)
- [SGLang (local models)](https://docs.openclaw.ai/ja-JP/providers/sglang)
- [StepFun](https://docs.openclaw.ai/ja-JP/providers/stepfun)
- [Synthetic](https://docs.openclaw.ai/ja-JP/providers/synthetic)
- [Tencent Cloud (TokenHub)](https://docs.openclaw.ai/ja-JP/providers/tencent)
- [Together AI](https://docs.openclaw.ai/ja-JP/providers/together)
- [Venice (Venice AI、プライバシー重視)](https://docs.openclaw.ai/ja-JP/providers/venice)
- [Vercel AI Gateway](https://docs.openclaw.ai/ja-JP/providers/vercel-ai-gateway)
- [vLLM (local models)](https://docs.openclaw.ai/ja-JP/providers/vllm)
- [Volcengine (Doubao)](https://docs.openclaw.ai/ja-JP/providers/volcengine)
- [Vydra](https://docs.openclaw.ai/ja-JP/providers/vydra)
- [xAI](https://docs.openclaw.ai/ja-JP/providers/xai)
- [Xiaomi](https://docs.openclaw.ai/ja-JP/providers/xiaomi)
- [Z.AI](https://docs.openclaw.ai/ja-JP/providers/zai)

## 共有概要ページ

- [追加のバンドル済みバリアント](https://docs.openclaw.ai/ja-JP/providers/models#additional-bundled-provider-variants) - Anthropic Vertex、Copilot Proxy、Gemini CLI OAuth
- [画像生成](https://docs.openclaw.ai/ja-JP/tools/image-generation) - 共有 `image_generate` ツール、プロバイダー選択、フェイルオーバー
- [音楽生成](https://docs.openclaw.ai/ja-JP/tools/music-generation) - 共有 `music_generate` ツール、プロバイダー選択、フェイルオーバー
- [動画生成](https://docs.openclaw.ai/ja-JP/tools/video-generation) - 共有 `video_generate` ツール、プロバイダー選択、フェイルオーバー

## 文字起こしプロバイダー

- [Deepgram (音声文字起こし)](https://docs.openclaw.ai/ja-JP/providers/deepgram)
- [ElevenLabs](https://docs.openclaw.ai/ja-JP/providers/elevenlabs#speech-to-text)
- [Mistral](https://docs.openclaw.ai/ja-JP/providers/mistral#audio-transcription-voxtral)
- [OpenAI](https://docs.openclaw.ai/ja-JP/providers/openai#speech-to-text)
- [SenseAudio](https://docs.openclaw.ai/ja-JP/providers/senseaudio)
- [xAI](https://docs.openclaw.ai/ja-JP/providers/xai#speech-to-text)

## コミュニティツール

- [Claude Max API Proxy](https://docs.openclaw.ai/ja-JP/providers/claude-max-api-proxy) - Claude サブスクリプション認証情報用のコミュニティプロキシ（使用前に Anthropic のポリシー/規約を確認してください）

完全なプロバイダーカタログ（xAI、Groq、Mistral など）と高度な設定については、 [モデルプロバイダー](https://docs.openclaw.ai/ja-JP/concepts/model-providers) を参照してください。