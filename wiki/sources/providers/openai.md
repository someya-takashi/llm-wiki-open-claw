---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/openai
source_path: raw/docs/providers/openai.md
doc_section: providers
title: "OpenAI"
ingested: 2026-06-14
tags: [provider, openai, gpt, codex, chatgpt, runtime, oauth]
related:
  - "[[providers/openai]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/agent-runtimes]]"
---

# OpenAI（解説）

> 原典: `raw/docs/providers/openai.md` ・ https://docs.openclaw.ai/ja-JP/providers/openai

## 一言まとめ

OpenAI は GPT モデルの開発者 API を提供し、Codex は ChatGPT プランのコーディングエージェントとして使える。OpenClaw は両サーフェスを**分離**し、`openai/*` を正規ルートとする。

## 位置づけ

[[providers/openai]] の詳細ソース。[[concepts/model-providers]] の主要プロバイダーかつ [[concepts/agent-runtimes]] の Codex ランタイム（[[sources/plugins/codex-harness]]）と密接。

## 仕組み・ふるまい

- **エージェントモデル**：`openai/*` の埋め込みエージェントターンは**既定でネイティブ Codex app-server ランタイム**経由。ChatGPT/Codex サブスクは Codex 認証でサインイン、または Codex 互換の OpenAI API キーバックアップ。
- **非エージェントサーフェス**：画像/埋め込み/音声/リアルタイムは直接 OpenAI API キーで（Codex 経由でない）。

## 設定・使い方の要点

- `OPENAI_API_KEY` または Codex OAuth（`openclaw models auth login`）。`agents.defaults.model.primary: "openai/gpt-5.5"`。`/fast` はネイティブ Responses で `service_tier=priority`。Codex 設定は [[sources/plugins/codex-harness]]。

## 注意点・落とし穴

- ⚠️ `openai/*` は既定で Codex ランタイムを通る（PI でなく）。`openai-codex/*` モデル ref は使わず正規 `openai/*` を使う（[[sources/concepts/model-providers]]）。

## 用語と略称

- **GPT** = OpenAI の汎用モデルファミリー
- **Codex** = OpenAI のコーディングエージェント/app-server
- **app-server ランタイム** = Codex のローカル実行サーバー
- **`service_tier`** = リクエスト優先度

## 関連ページ

- [[providers/openai]] — 対応するプロバイダーページ
- [[concepts/model-providers]] / [[concepts/agent-runtimes]] / [[sources/plugins/codex-harness]] / [[concepts/oauth]]
