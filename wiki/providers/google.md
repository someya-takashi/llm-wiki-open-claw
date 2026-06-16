---
type: provider
aliases: [Google, Gemini]
tags: [model-provider, llm, google, gemini, multimodal]
related:
  - "[[concepts/model-providers]]"
  - "[[concepts/media-understanding]]"
sources:
  - "[[sources/providers/google]]"
updated: 2026-06-14
---

# Google (Gemini)

**Google** は Google AI Studio 経由で **Gemini** モデル（`google/*`）を提供する多機能プロバイダー。チャットに加え、**画像生成・メディア理解・TTS・Web 検索（Gemini Grounding）** を 1 プロバイダーで担う。[[concepts/model-providers]] の主要プロバイダーで、[[concepts/media-understanding]]・[[concepts/voice]]・[[concepts/web-search]]（[[sources/tools/gemini-search]]）にも横断的に関わる。認証は `GEMINI_API_KEY`/`GOOGLE_API_KEY`。

詳細は [[sources/providers/google]]。

## 関連

- [[concepts/model-providers]] / [[concepts/media-understanding]] / [[concepts/voice]] / [[concepts/web-search]]
- [[providers/anthropic]] / [[providers/openai]] / [[components/gateway]]
