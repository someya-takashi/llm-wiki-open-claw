---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/openai-http-api
source_path: raw/docs/gateway/openai-http-api.md
doc_section: gateway
title: "OpenAI チャット補完"
ingested: 2026-06-14
tags: [http-api, openai-compat, chat-completions, agent-target, sse]
related:
  - "[[concepts/http-api]]"
  - "[[components/gateway]]"
  - "[[concepts/agent]]"
---

# OpenAI チャット補完（解説）

> 原典: `raw/docs/gateway/openai-http-api.md` ・ https://docs.openclaw.ai/ja-JP/gateway/openai-http-api

## 一言まとめ

Gateway が出せる **OpenAI 互換の `POST /v1/chat/completions`**（＋`/v1/models`・`/v1/embeddings`）。**既定で無効**。中身は通常の Gateway エージェント実行（`openclaw agent` と同じコードパス）なので、ルーティング・権限・設定は Gateway と一致。

## 位置づけ

[[concepts/http-api]] の中核エンドポイント。Open WebUI / LobeChat / LibreChat 等の**セルフホスト UI から OpenClaw のエージェントを OpenAI クライアントとして叩く**ための互換面。よりエージェントネイティブな用途は [[sources/gateway/openresponses-http-api]]。

## 仕組み・ふるまい

- **エージェント優先のモデル契約**：OpenAI の `model` は生のプロバイダーモデル ID ではなく**エージェントターゲット**。`openclaw`／`openclaw/default`＝既定エージェント、`openclaw/<agentId>`＝特定エージェント。`/v1/models` は（プロバイダーモデルでもサブエージェントでもなく）トップレベルのエージェントターゲットを返す。
- ヘッダー上書き：`x-openclaw-model`（バックエンドモデル）・`x-openclaw-agent-id`・`x-openclaw-session-key`（セッション完全制御）・`x-openclaw-message-channel`（合成入口チャネル文脈）。
- **セッション**：既定は**リクエストごとステートレス**（毎回新キー）。OpenAI の `user` 文字列があれば安定キーを導出して再利用。
- **ストリーミング**：`stream: true` で SSE（`data: <json>` 行、`data: [DONE]` 終端）。
- **ツール契約**：`tools`（function サブセット）・`tool_choice: "auto"|"none"`・`role:"tool"` フォロー・`tool_call_id`。`finish_reason:"tool_calls"` でツール呼び出しを返し、結果を返して推論ループ継続。未対応バリアントは `400 invalid_request_error`（`tool_choice:"required"` は未強制）。

## 設定・使い方の要点

- 有効化：`gateway.http.endpoints.chatCompletions.enabled: true`（WS と同じポートで多重化）。
- 認証は Gateway 認証モード（token/password→`Authorization: Bearer`、trusted-proxy、none）。
- Open WebUI：ベース URL `http://127.0.0.1:18789/v1`、API キー＝Gateway bearer、モデル `openclaw/default`。
- `max_completion_tokens`（現行名）＞ `max_tokens`（レガシーエイリアス）。上流ワイヤー名はプロバイダートランスポートが選択。

## 注意点・落とし穴

- ⚠️ **完全なオペレーターアクセスサーフェス**：有効な Gateway トークン/パスワードは所有者資格情報相当。ユーザー別の狭いスコープ境界は無い。共有シークレットモードでは呼び出し元の狭い `x-openclaw-scopes` を無視して**フルのオペレーター既定**（`operator.admin/approvals/pairing/read/talk.secrets/write`）を復元し、チャットターンを所有者送信として扱う。
- **loopback/tailnet/プライベート入口のみ**に置く。公開インターネットへ直接晒さない。

## 用語と略称

- **OpenAI 互換 / Chat Completions** = OpenAI の `/v1/chat/completions` 形式
- **エージェントターゲット** = `model` 値が指す OpenClaw エージェント
- **SSE** = Server-Sent Events（逐次プッシュ）
- **RAG** = Retrieval-Augmented Generation（検索拡張生成、`/v1/embeddings` を使う）

## 関連ページ

- [[concepts/http-api]] — 対応する概念ページ
- [[sources/gateway/openresponses-http-api]] / [[sources/gateway/tools-invoke-http-api]]
- [[components/gateway]] / [[concepts/agent]] / [[concepts/authentication]] / [[providers/openai]]
