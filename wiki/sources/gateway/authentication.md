---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/authentication
source_path: raw/docs/gateway/authentication.md
doc_section: gateway
title: "認証（モデルプロバイダー）"
ingested: 2026-06-14
tags: [authentication, model-auth, api-key, oauth, auth-profiles, credentials]
related:
  - "[[concepts/authentication]]"
  - "[[concepts/oauth]]"
  - "[[concepts/secrets]]"
---

# 認証（モデルプロバイダー）（解説）

> 原典: `raw/docs/gateway/authentication.md` ・ https://docs.openclaw.ai/ja-JP/gateway/authentication
>
> ⚠️ このページは**モデルプロバイダー認証**（API キー・OAuth・Claude CLI 再利用・setup-token）のリファレンス。**Gateway 接続認証**（token/password/trusted-proxy）は [[sources/gateway/configuration]] と [[sources/gateway/trusted-proxy-auth]] を見ること。

## 一言まとめ

エージェントがモデルを呼ぶための**認証情報の設定・確認・制御**を扱うページ。長期稼働の Gateway ホストでは **API キー**が最も予測しやすく、サブスク/OAuth（[[concepts/oauth]]）や Anthropic の Claude CLI 再利用も使える。

## 位置づけ

[[concepts/authentication]] の「モデル認証」面（3 つある認証サーフェスのうちの 1 つ）。OAuth/サブスクの詳細は [[concepts/oauth]]、秘密値の保存は [[concepts/secrets]]（SecretRef）、適格性/理由コードの正規定義は [[sources/auth-credential-semantics]]。

## 仕組み・ふるまい

- **推奨は API キー**：プロバイダーコンソールで作成 → **Gateway ホスト**に置く（`export <PROVIDER>_API_KEY=...`）。systemd/launchd 下では `~/.openclaw/.env` に置いてデーモンが読めるように。確認は `openclaw models status` / `openclaw doctor`。
- **`auth-profiles.json`**（認証情報のみ保存、正規形 `{ version, profiles }`）：`api_key`/`token` プロファイル。エンドポイント詳細（`baseUrl`/`api`/モデル ID/ヘッダー）は **`auth-profiles.json` ではなく** `openclaw.json` の `models.providers.<id>` に置く。レガシーなフラット形は `openclaw doctor --fix` で正規化（`.legacy-flat.*.bak` を残す）。
- **Anthropic**：Claude CLI 再利用が再びサポート（`claude auth login` → `openclaw models auth login --provider anthropic --method cli --set-default`）。setup-token も可。API キーが最も予測しやすい。
- **SecretRef 対応**：`api_key` は `keyRef`、`token` は `tokenRef`（`{ source, provider, id }`）。⚠️ `mode: "oauth"` のプロファイルは SecretRef を**拒否**（→ [[sources/auth-credential-semantics]] の OAuth ガード）。
- **API キーのローテーション**：レート制限（429/quota 等）時のみ代替キーで再試行。優先順 `OPENCLAW_LIVE_<PROVIDER>_KEY` → `<PROVIDER>_API_KEYS` → `<PROVIDER>_API_KEY` → `<PROVIDER>_API_KEY_*`（重複排除）。レート制限以外では切り替えない。

## 設定・使い方の要点

- 確認：`openclaw models status [--check|--probe]`（`--check` は終了コード 1=欠落/期限切れ・2=期限切れ間近）。プローブ行は認証プロファイル/env/`models.json` 由来。
- **使う認証情報の制御**：セッションは `/model <alias>@<profileId>`（例 `Opus@anthropic:work`）、エージェントは `openclaw models auth order get/set/clear --provider <p> [--agent <id>]`（`auth-state.json` に保存）。

## 注意点・落とし穴

- 「No credentials found」→ Gateway ホストに API キー or setup-token を置いて `openclaw models status` で再確認。
- `excluded_by_auth_order`（明示的な `auth.order` が保存プロファイルを省いている）・`no_model`（認証はあるがプローブ可能モデルが無い）はプローブの理由コード（→ [[sources/auth-credential-semantics]]）。
- レート制限クールダウンは**モデル単位**のことがある（同プロバイダーの別モデルはまだ使える）。
- Bedrock の `aws-sdk` など外部認証ルートは「認証情報」ではない → `auth.profiles.<id>.mode: "aws-sdk"`（`auth-profiles.json` に `type: "aws-sdk"` を書かない）。

## 用語と略称

- **モデル認証** = エージェントがモデルプロバイダーを呼ぶための認証（接続認証とは別）
- **auth-profiles.json** = 認証情報（API キー/トークン）を保存するエージェントごとのストア
- **setup-token** = Anthropic のトークン取得フロー
- **keyRef / tokenRef** = API キー/トークンを SecretRef で参照するフィールド
- **probe（プローブ）** = 認証情報がモデルに通るかのライブ確認

## 関連ページ

- [[concepts/authentication]] — 認証の全体像（3 サーフェス）
- [[concepts/oauth]] / [[concepts/secrets]]
- [[sources/auth-credential-semantics]] / [[sources/gateway/trusted-proxy-auth]] / [[sources/gateway/secrets]]
