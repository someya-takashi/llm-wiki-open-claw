---
type: concept
aliases: [OAuth, subscription auth, サブスクリプション認証]
tags: [oauth, auth, pkce, credentials, multi-account]
related:
  - "[[concepts/authentication]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/agent-workspace]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/multi-agent]]"
sources:
  - "[[sources/concepts/oauth]]"
updated: 2026-06-14
---

# OAuth（モデル認証）

OpenClaw は対応プロバイダー向けに **OAuth による「サブスクリプション認証」**（OpenAI Codex の ChatGPT OAuth、Anthropic の Claude CLI/サブスク認証など）をサポートする。トークンは `~/.openclaw/agents/<id>/agent/auth-profiles.json` に集約保存される。これは [[concepts/authentication]] の 3 レイヤーのうち**モデル認証**の OAuth/サブスク部分に当たる（API キー側や接続認証とは別）。

## なぜ重要か

サブスク認証は便利だが、**複数ツールで二重ログインするとリフレッシュトークンのローテーションで片方が落ちる**という固有の罠がある。OpenClaw は `auth-profiles.json` を**トークンシンク**（認証情報を 1 か所から読む）として扱い、複数プロファイルを決定的にルーティングしてこれを緩和する。期限切れトークンはファイルロック下で自動更新され、継承プロファイルの更新はメインエージェントストアへ書き戻す。

## 押さえる点

- 交換フロー：Anthropic は setup-token、OpenAI Codex は **PKCE**（`authorize` → `127.0.0.1:1455/auth/callback` → `token` 交換 → `{access, refresh, expires, accountId}` 保存）。
- 認証情報は [[concepts/agent-workspace]] に**置かない**（`~/.openclaw/` 側）。Gateway 接続の認証（[[concepts/pairing]]・`gateway.auth.*`）とは別レイヤー。
- **複数アカウント**：推奨は個別エージェント（[[concepts/multi-agent]]）で分離、高度には 1 エージェント内の複数プロファイル（`/model Opus@anthropic:work`）。
- 本番 Anthropic は **API キー認証がより安全な推奨パス**。

ログインコマンド・ストレージ詳細・継承は [[sources/concepts/oauth]]。

## 関連

- [[concepts/agent-runtimes]] — Codex/Claude CLI 等の実行バックエンド
- [[concepts/agent-workspace]] — 認証を置かないワークスペース
- [[providers/openai]] / [[providers/anthropic]] / [[concepts/model-providers]]
