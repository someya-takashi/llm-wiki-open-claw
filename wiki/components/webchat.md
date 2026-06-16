---
type: component
aliases: [WebChat, ウェブチャット]
tags: [webchat, chat-ui, swiftui, gateway-ws, client]
concepts:
  - "[[concepts/architecture]]"
  - "[[concepts/session]]"
  - "[[concepts/messages]]"
related:
  - "[[components/gateway]]"
  - "[[components/control-ui]]"
sources:
  - "[[sources/web/webchat]]"
updated: 2026-06-14
---

# WebChat

**WebChat** は、[[components/gateway]] 用の**ネイティブチャット UI**（macOS/iOS の SwiftUI）。埋め込みブラウザもローカル静的サーバも介さず、**Gateway WS と直接**話す軽量クライアント。返信は常に WebChat に戻る（決定的ルーティング）。[[components/control-ui]]（ブラウザー）・[[components/cli]]（端末）と同じ [[concepts/architecture]] の WS クライアントで、他チャネルと**同じ [[concepts/session]] とルーティング**（[[concepts/messages]]）を共有する。

## 仕組み

- `chat.history`/`chat.send`/`chat.inject` を使い、履歴は常に Gateway から取得（ローカル監視なし、到達不能なら読み取り専用）。
- **2 つのデータパス**：①永続的なセッション JSONL トランスクリプト（モデル/ランタイムの真実）②Gateway の `ReplyPayload` ライブ配信イベント（表示向け正規化）。WebChat は通常の Pi ターン外の表示メッセージ（`chat.inject`・非エージェント返信・中止部分・メディア補足）のみ注入する。
- 表示正規化で配信ディレクティブタグ・ツール呼び出し XML・制御トークン・思考専用ペイロード（`isReasoning`）・`NO_REPLY` を除去（[[concepts/messages]] のサイレント返信と整合）。長い会話の区切りは [[concepts/compaction]] エントリとして表示。

## 位置づけ

詳細（トランスクリプト/配信モデル・設定キー `gateway.webchat.chatHistoryMaxChars`）は [[sources/web/webchat]]。リモートは WS を SSH/Tailscale でトンネルするだけで、別 WebChat サーバは不要（[[concepts/remote-access]]）。

## 関連

- [[concepts/architecture]] / [[concepts/session]] / [[concepts/messages]] / [[concepts/compaction]]
- [[components/gateway]] / [[components/control-ui]] / [[components/cli]]
- [[sources/web/webchat]]
