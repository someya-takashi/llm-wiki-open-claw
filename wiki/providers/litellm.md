---
type: provider
aliases: [LiteLLM]
tags: [model-provider, llm, litellm, gateway, unified-api]
related:
  - "[[concepts/model-providers]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/providers/litellm]]"
updated: 2026-06-14
---

# LiteLLM

**LiteLLM** は、100 以上のプロバイダーに統一 API を提供する OSS の LLM Gateway。OpenClaw からは OpenAI 互換のカスタムプロバイダー（`models.providers.<id>` に LiteLLM の URL を baseUrl として登録）として扱い、実バックエンドの振り分け・コスト追跡・ロギングを LiteLLM 側に一元化できる。[[concepts/model-providers]] の「統合 Gateway」系。

詳細・設定は [[sources/providers/litellm]]。

## 関連

- [[concepts/model-providers]] / [[concepts/local-models]] / [[concepts/model-failover]] / [[components/gateway]]
