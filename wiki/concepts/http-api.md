---
type: concept
aliases: [HTTP API, OpenAI 互換 API, OpenAI-compatible API]
tags: [http-api, openai-compat, chat-completions, openresponses, tools-invoke]
related:
  - "[[concepts/architecture]]"
  - "[[concepts/agent]]"
  - "[[concepts/authentication]]"
  - "[[concepts/security]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/openai-http-api]]"
  - "[[sources/gateway/openresponses-http-api]]"
  - "[[sources/gateway/tools-invoke-http-api]]"
updated: 2026-06-14
---

# HTTP API（OpenAI 互換サーフェス）

HTTP API は、[[components/gateway]] が WS（WebSocket）制御プレーンと**同じポートで多重化**して出せる HTTP エンドポイント群。WS が OpenClaw ネイティブのクライアント（[[concepts/architecture]]）向けなのに対し、こちらは**既存の OpenAI 互換クライアント／ツールから OpenClaw を叩く**ための互換面。3 つのエンドポイントがある：

| エンドポイント | 既定 | 用途 | ソース |
|---|---|---|---|
| `POST /v1/chat/completions` | **無効** | OpenAI Chat Completions 互換（＋`/v1/models`・`/v1/embeddings`）。Open WebUI/LobeChat/LibreChat 等 | [[sources/gateway/openai-http-api]] |
| `POST /v1/responses` | **無効** | OpenResponses 互換（項目ベース入力・画像/ファイル等のマルチモーダル） | [[sources/gateway/openresponses-http-api]] |
| `POST /tools/invoke` | **常時有効** | 単一ツールを推論を介さず直接実行 | [[sources/gateway/tools-invoke-http-api]] |

## 共通の設計

- **エージェント実行と同じコードパス**：チャット系の中身は通常の Gateway エージェント実行（`openclaw agent` と同じ）なので、ルーティング・権限・設定が Gateway と一致する。
- **エージェント優先のモデル契約**：OpenAI の `model` フィールドを生のプロバイダーモデル ID ではなく**エージェントターゲット**として解釈（`openclaw`/`openclaw/default`/`openclaw/<agentId>`）。バックエンドモデルは `x-openclaw-model` ヘッダーで上書き。これが [[concepts/agent]] とのつながり——HTTP クライアントから見える「モデル」は実は OpenClaw のエージェント。
- **認証は Gateway 認証モード**（[[concepts/authentication]]）：token/password → `Authorization: Bearer`、trusted-proxy、none。

## セキュリティ境界（最重要）

⚠️ これらは**完全なオペレーターアクセスサーフェス**として扱う。狭いユーザー別スコープモデルでは**ない**：

- 有効な Gateway トークン/パスワードは**所有者/オペレーター資格情報相当**。
- 共有シークレットモード（token/password）では、呼び出し元が狭い `x-openclaw-scopes` を送っても**無視**してフルのオペレーター既定（`operator.admin/approvals/pairing/read/talk.secrets/write`）を復元し、ターンを所有者送信として扱う。
- `/tools/invoke` は既定で **RCE 面（`exec`/`spawn`/`shell`）・任意ファイル変更（`fs_write` 等）・`gateway`/`nodes`/`cron` をハード拒否**。Exec 承認はこの HTTP 面の認可境界ではない。
- いずれも **loopback/tailnet/プライベート入口のみ**に置き、公開インターネットへ直接晒さない（[[concepts/security]]・[[concepts/sandboxing]]）。

## 代表ソース

- [[sources/gateway/openai-http-api]] — Chat Completions（エージェント契約・SSE・ツール契約・Open WebUI）
- [[sources/gateway/openresponses-http-api]] — Responses（項目入力・マルチモーダル・URL フェッチガード）
- [[sources/gateway/tools-invoke-http-api]] — 単一ツール直接呼び出しと拒否リスト

## 関連ページ

- [[concepts/architecture]] / [[concepts/agent]] / [[concepts/session-tool]]
- [[concepts/authentication]] / [[concepts/security]] / [[components/gateway]]
