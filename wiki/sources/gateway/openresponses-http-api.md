---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/openresponses-http-api
source_path: raw/docs/gateway/openresponses-http-api.md
doc_section: gateway
title: "OpenResponses API"
ingested: 2026-06-14
tags: [http-api, openresponses, responses, items, multimodal, sse]
related:
  - "[[concepts/http-api]]"
  - "[[sources/gateway/openai-http-api]]"
  - "[[components/gateway]]"
---

# OpenResponses API（解説）

> 原典: `raw/docs/gateway/openresponses-http-api.md` ・ https://docs.openclaw.ai/ja-JP/gateway/openresponses-http-api

## 一言まとめ

Gateway が出せる **OpenResponses 互換の `POST /v1/responses`**（**既定で無効**）。Chat Completions より新しい「項目（item）ベース入力」のエージェントネイティブな面で、認証・セキュリティ・ルーティングは [[sources/gateway/openai-http-api]] と一致。

## 位置づけ

[[concepts/http-api]] のもう 1 つの主要エンドポイント。エージェント優先のモデル契約・`openclaw/default`・埋め込みパススルー・バックエンドモデル上書きは Chat Completions と共通。本ページ固有は**項目入力・マルチモーダル入力（画像/ファイル）・レスポンスイベント**。

## 仕組み・ふるまい

- **リクエスト形状**：`input`（文字列 or 項目配列）・`instructions`（システムプロンプトにマージ）・`tools`/`tool_choice`・`stream`・`max_output_tokens`・`user`（安定セッション）。`previous_response_id` で同一スコープ内のセッション再利用。`max_tool_calls`/`reasoning`/`metadata`/`store`/`truncation` は受理するが**現状無視**。
- **項目**：`message`（`system`/`developer`→システムプロンプト追加、最新 `user`＝現在メッセージ）、`function_call_output`（`call_id`＋`output` でツール結果返却）、`reasoning`/`item_reference`（互換受理・無視）。
- **マルチモーダル**：`input_image`（base64/URL、JPEG/PNG/GIF/WebP/HEIC/HEIF、最大 10MB、HEIC/HEIF は JPEG 正規化）、`input_file`（text 系/JSON/PDF、最大 5MB）。⚠️ **ファイル内容はユーザーメッセージでなくシステムプロンプトに一時挿入され、`<<<EXTERNAL_UNTRUSTED_CONTENT>>>` 境界マーカーで「信頼されない外部データ」としてラップ**（命令ではなくデータ扱い）。PDF はまずテキスト抽出、不足なら先頭ページを画像化（`document-extract` Plugin / `pdfjs-dist`）。
- **URL フェッチ**：既定 `allowUrl: true`、`maxUrlParts: 8`、DNS 解決・プライベート IP ブロック・リダイレクト上限・タイムアウトでガード、ホスト名許可リスト（`urlAllowlist`、ワイルドカードサブドメイン可）。
- **ストリーミング**：`response.created`→`output_text.delta`→`response.completed`（エラー時 `response.failed`）の SSE イベント列、`data: [DONE]` 終端。`usage` は OpenAI 形式エイリアスに正規化。

## 設定・使い方の要点

- 有効化：`gateway.http.endpoints.responses.enabled: true`。同じ面に `/v1/models`・`/v1/embeddings`・`/v1/chat/completions` も含まれる。
- ファイル/画像の上限は `gateway.http.endpoints.responses.files`/`images`（`maxBodyBytes` 既定 20MB 等）で調整。

## 注意点・落とし穴

- 認証・セキュリティ境界は Chat Completions と同一（**完全オペレーターアクセス**、共有シークレットは狭い `x-openclaw-scopes` を無視）。
- ⚠️ ホスト名許可リストに足しても**プライベート/内部 IP ブロックはバイパスされない**。公開 Gateway ではネットワーク外向き制御も併用（SSRF 対策）。

## 用語と略称

- **OpenResponses** = OpenAI Responses API 互換の項目ベース契約
- **項目（item）** = `message`/`function_call_output` 等の入力単位
- **SSRF** = Server-Side Request Forgery（サーバーを踏み台にした内部リクエスト）
- **HEIC/HEIF** = Apple の画像フォーマット

## 関連ページ

- [[concepts/http-api]] — 対応する概念ページ
- [[sources/gateway/openai-http-api]] / [[sources/gateway/tools-invoke-http-api]]
- [[components/gateway]] / [[concepts/security]]
