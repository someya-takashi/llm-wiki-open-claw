---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/anthropic
source_path: raw/docs/providers/anthropic.md
doc_section: providers
title: "Anthropic"
ingested: 2026-06-14
tags: [provider, anthropic, claude, api-key, claude-cli, oauth]
related:
  - "[[providers/anthropic]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/oauth]]"
---

# Anthropic（解説）

> 原典: `raw/docs/providers/anthropic.md` ・ https://docs.openclaw.ai/ja-JP/providers/anthropic

## 一言まとめ

Anthropic は **Claude** モデルファミリー（`anthropic/*`）を提供。OpenClaw は 2 つの認証ルート——**API キー**（使用量課金）と **Claude CLI**（同ホストの既存 Claude ログイン再利用）——をサポートする。

## 位置づけ

[[providers/anthropic]] の詳細ソース。[[concepts/model-providers]] の主要組み込みプロバイダー。Claude CLI 認証は [[concepts/oauth]]・[[sources/gateway/cli-backends]] と関連。

## 仕組み・ふるまい

- **API キー**：`anthropic/*` モデルへ直接アクセス（使用量ベース課金）。
- **Claude CLI**：同ホストの `claude` ログインを再利用（サブスク利用）。`claude-cli` バックエンド（[[sources/gateway/cli-backends]]）でも使う。
- `anthropic-vertex`：Google Vertex 上の Anthropic を Vertex 認証情報がある場合に暗黙サポート。

## 設定・使い方の要点

- `ANTHROPIC_API_KEY` または `openclaw models auth login --provider anthropic`。`agents.defaults.model.primary: "anthropic/claude-opus-4-6"` 等。`/fast` は公開 Anthropic で `service_tier` にマップ（[[sources/tools/slash-commands]]）。

## 用語と略称

- **Claude** = Anthropic のモデルファミリー
- **Claude CLI** = Anthropic 公式 CLI（ログイン再利用）
- **Vertex** = Google Cloud の AI プラットフォーム
- **`service_tier`** = リクエストの優先度ティア

## 関連ページ

- [[providers/anthropic]] — 対応するプロバイダーページ
- [[concepts/model-providers]] / [[concepts/oauth]] / [[sources/gateway/cli-backends]]
