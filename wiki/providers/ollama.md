---
type: provider
aliases: [Ollama]
tags: [model-provider, llm, ollama, local-models, self-hosted]
related:
  - "[[concepts/model-providers]]"
  - "[[concepts/local-models]]"
sources:
  - "[[sources/providers/ollama]]"
updated: 2026-06-14
---

# Ollama

**Ollama** は、ローカル/セルフホストとクラウドの LLM をネイティブ API（`/api/chat`）で扱うプロバイダー（`ollama/*`）。`Cloud + Local`・`Cloud only`（`ollama.com`）・`Local only` の 3 モードを持つ。[[concepts/local-models]] の代表的バックエンドで、Web 検索（[[sources/tools/ollama-search]]）やメモリ埋め込み（[[sources/plugins/memory-lancedb]]）にも使える。`models.mode: "merge"` でホスト型プロバイダーと併存する。

詳細・モード設定は [[sources/providers/ollama]]。

## 関連

- [[concepts/model-providers]] / [[concepts/local-models]]
- [[sources/tools/ollama-search]] / [[sources/plugins/memory-lancedb]] / [[components/gateway]]
