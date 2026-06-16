---
type: provider
aliases: [Anthropic, Claude]
tags: [model-provider, llm, anthropic, claude]
related:
  - "[[concepts/model-providers]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/providers/anthropic]]"
updated: 2026-06-14
---

# Anthropic

**Anthropic** は **Claude** モデルファミリー（`anthropic/*`）を提供するプロバイダー。[[concepts/model-providers]] の主要組み込みプロバイダーで、2 つの認証ルートを持つ：**API キー**（使用量課金）と **Claude CLI**（同ホストの既存ログイン再利用、[[sources/gateway/cli-backends]] の `claude-cli` でも使用）。`anthropic-vertex` で Google Vertex 上の Claude も暗黙サポート。

詳細・認証手順は [[sources/providers/anthropic]]。

## 関連

- [[concepts/model-providers]] / [[concepts/oauth]] / [[components/gateway]]
- [[providers/openai]] / [[providers/google]] / [[sources/gateway/cli-backends]]
