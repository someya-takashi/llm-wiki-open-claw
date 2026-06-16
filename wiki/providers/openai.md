---
type: provider
aliases: [OpenAI, GPT, Codex]
tags: [model-provider, llm, openai, gpt, codex]
related:
  - "[[concepts/model-providers]]"
  - "[[concepts/agent-runtimes]]"
sources:
  - "[[sources/providers/openai]]"
updated: 2026-06-14
---

# OpenAI

**OpenAI** は GPT モデル（`openai/*`）の開発者 API と、ChatGPT プランの **Codex** コーディングエージェントを提供する。[[concepts/model-providers]] の主要プロバイダーであり、`openai/*` の埋め込みエージェントターンは**既定でネイティブ Codex app-server ランタイム**を通る（[[concepts/agent-runtimes]] / [[sources/plugins/codex-harness]]）。画像/埋め込み/音声/リアルタイムは直接 API キーで使える。

詳細・Codex 認証は [[sources/providers/openai]]。

## 関連

- [[concepts/model-providers]] / [[concepts/agent-runtimes]] / [[concepts/oauth]]
- [[sources/plugins/codex-harness]] / [[providers/anthropic]] / [[components/gateway]]
