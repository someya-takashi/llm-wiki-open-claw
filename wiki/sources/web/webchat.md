---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/web/webchat
source_path: raw/docs/web/webchat.md
doc_section: web
title: "WebChat"
ingested: 2026-06-14
tags: [webchat, chat-ui, gateway-ws, transcript, chat-history]
related:
  - "[[components/webchat]]"
  - "[[concepts/session]]"
  - "[[concepts/messages]]"
---

# WebChat（解説）

> 原典: `raw/docs/web/webchat.md` ・ https://docs.openclaw.ai/ja-JP/web/webchat

## 一言まとめ

Gateway 用の**ネイティブチャット UI**（macOS/iOS の SwiftUI）。埋め込みブラウザもローカル静的サーバも要らず、**Gateway WS と直接**話す。返信は常に WebChat に戻る（決定的ルーティング）。

## 位置づけ

[[components/webchat]] の中核ソース。[[components/control-ui]]（ブラウザー）や [[components/cli]]（端末）と同じく、他チャネルと**同じ [[concepts/session]] とルーティング**を使う Gateway クライアント（[[concepts/messages]]）。

## 仕組み・ふるまい

- UI は `chat.history`/`chat.send`/`chat.inject` を使う。履歴は常に Gateway から取得（ローカルファイル監視なし）、到達不能なら**読み取り専用**。
- **2 つのデータパス**：①セッション JSONL（永続的なモデル/ランタイムトランスクリプト、Pi が `user`/`assistant`/`toolResult` を書く）②Gateway の `ReplyPayload` イベント（ライブ配信プロジェクション、表示向けに正規化）。WebChat は通常の Pi ターン外の表示メッセージ（`chat.inject`・非エージェントコマンド返信・中止部分・メディア補足）のみトランスクリプトに注入。
- **表示正規化**：`chat.history` は配信ディレクティブタグ（`reply_to_*`/`audio_as_voice`）・ツール呼び出し XML・制御トークンを除去し、`NO_REPLY` のみのエントリや `isReasoning: true`（思考専用）を表示/音声から除外。Compaction エントリは区切りとして表示しチェックポイントへリンク。

## 設定・使い方の要点

- `gateway.webchat.chatHistoryMaxChars`（履歴のテキスト上限、リクエスト単位 `maxChars` で上書き可）。関連：`gateway.port`/`bind`・`gateway.auth.*`・`gateway.remote.*`・`session.*`。
- リモートは WS を SSH/Tailscale でトンネルするだけ（別 WebChat サーバ不要、[[concepts/remote-access]]）。

## 注意点・落とし穴

- `chat.history` はサイズ制限（長文切り詰め・`[chat.history omitted: message too large]`）。中止実行は部分テキストを中止メタデータ付きで永続化。Control UI は裏の `sessionId` を記憶して再接続/更新後も会話継続。

## 用語と略称

- **WebChat** = Gateway 直結のネイティブチャット UI
- **SwiftUI** = Apple のネイティブ UI フレームワーク
- **トランスクリプト（transcript）** = セッションの会話記録（JSONL）
- **`ReplyPayload`** = ライブ配信用に正規化された返信イベント
- **Compaction** = 長い会話の要約圧縮

## 関連ページ

- [[components/webchat]] — 対応する構成要素ページ
- [[sources/web/control-ui]] / [[sources/web/tui]]
- [[concepts/session]] / [[concepts/messages]] / [[concepts/compaction]]
