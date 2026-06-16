---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/oauth
source_path: raw/docs/concepts/oauth.md
doc_section: concepts
title: "OAuth"
ingested: 2026-06-14
tags: [oauth, auth, pkce, credentials, anthropic, codex, multi-account]
related:
  - "[[concepts/oauth]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/agent-workspace]]"
---

# OAuth（解説）

> 原典: `raw/docs/concepts/oauth.md` ・ https://docs.openclaw.ai/ja-JP/concepts/oauth

## 一言まとめ

OpenClaw は対応プロバイダー向けに **OAuth による「サブスクリプション認証」**（特に OpenAI Codex の ChatGPT OAuth、Anthropic の Claude CL/サブスク認証）をサポートする。トークン交換（PKCE）の仕組み、トークンの保存場所と理由（トークンシンク）、複数アカウントの扱い方を説明したページ。

## 位置づけ

モデルを呼ぶための**認証情報まわり**。実行バックエンドの選択 [[concepts/agent-runtimes]]（Codex/Claude CLI など）と表裏で、認証は [[concepts/agent-workspace]] に含めない `~/.openclaw/agents/<id>/agent/auth-profiles.json` に保存される。Gateway 接続そのものの認証（[[concepts/pairing]] や `gateway.auth.*`）とは別レイヤーである点に注意。

## 仕組み・ふるまい

### トークンシンク（なぜ集約するか）

OAuth プロバイダーはログイン/更新のたびに新しいリフレッシュトークンを発行し、古いものを無効化することがある。OpenClaw と Claude Code/Codex CLI の両方でログインすると、後でどちらかがランダムに「ログアウト」する症状が起きる。これを抑えるため OpenClaw は `auth-profiles.json` を**トークンシンク**（認証情報を 1 か所から読む）として扱い、複数プロファイルを保持して決定的にルーティングする。

### ストレージ

- 認証プロファイル（OAuth＋API キー＋値レベル参照）：`~/.openclaw/agents/<agentId>/agent/auth-profiles.json`。
- レガシー互換：`auth.json`（静的 `api_key` は検出時消去）、インポート専用 `~/.openclaw/credentials/oauth.json`（初回使用時に取り込み）。`$OPENCLAW_STATE_DIR` を尊重。
- セカンダリエージェントにローカルプロファイルが無ければ、メイン/デフォルトエージェントストアから**読み取りスルー継承**（複製はしない。リフレッシュトークンは機密のため通常コピーをスキップ）。

### OAuth 交換（ログインの仕組み）

対話ログインは `@earendil-works/pi-ai` 実装。

- **Anthropic setup-token**：OpenClaw から開始 or paste-token → 認証プロファイルに保存 → モデル選択は `anthropic/...` のまま。
- **OpenAI Codex（ChatGPT OAuth, PKCE）**：PKCE verifier/challenge＋`state` 生成 → `auth.openai.com/oauth/authorize` を開く → `127.0.0.1:1455/auth/callback` で捕捉（無理ならリダイレクト URL/コードを貼付）→ `oauth/token` で交換 → `{access, refresh, expires, accountId}` を保存。ウィザードは `openclaw onboard` → 認証選択 `openai-codex`。

### 更新＋有効期限

プロファイルは `expires` を保存。未来なら保存トークンを使用、期限切れならファイルロック下で更新して上書き。継承プロファイルの更新は**メインエージェントストアへ書き戻す**。一部の外部 CLI 認証は外部管理のままで、OpenClaw はその CLI 認証ストアを再読み取りする。

## 設定・使い方の要点

- 汎用ログイン：`openclaw models auth login --provider <id>`。provider plugin が独自の OAuth/API キーフローを同梱できる。
- **複数アカウント**：①推奨＝個別エージェント（`openclaw agents add work` / `personal` でセッション・認証・ワークスペースを分離）、②高度＝1 エージェント内の複数プロファイル（`auth.order` でグローバル指定、`/model Opus@anthropic:work` でセッション上書き）。プロファイル ID は `openclaw channels list --json` の `auth[]` で確認。

## 注意点・落とし穴

- **本番の Anthropic は API キー認証がより安全な推奨パス**。Claude CLI 再利用・`claude -p` は、Anthropic スタッフの言により現状認可済みとして扱うが、ポリシーは変わり得る。
- リフレッシュトークンは特に機密。外部 CLI と二重ログインするとトークンローテーションで片方が落ちる（トークンシンクで緩和）。

## 用語と略称

- **OAuth** = 認証情報を安全に委譲する認可規格
- **PKCE** = Proof Key for Code Exchange（公開クライアント向けに認可コード横取りを防ぐ OAuth 拡張）
- **リフレッシュトークン** = アクセストークンを再発行するための長期トークン
- **トークンシンク** = 認証情報を 1 か所に集約して読む方式
- **setup-token** = Anthropic のトークン取得フロー

## 関連ページ

- [[concepts/oauth]] — 対応する概念ページ
- [[concepts/agent-runtimes]] — Codex/Claude CLI 等の実行バックエンド
- [[concepts/agent-workspace]] — 認証情報を**置かない**ワークスペース
- [[providers/openai]] / [[providers/anthropic]] / [[concepts/model-providers]]
