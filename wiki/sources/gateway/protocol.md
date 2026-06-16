---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/protocol
source_path: raw/docs/gateway/protocol.md
doc_section: gateway
title: "Gateway プロトコル"
ingested: 2026-06-14
tags: [protocol, websocket, rpc, roles, scopes, handshake, contract]
related:
  - "[[concepts/architecture]]"
  - "[[components/gateway]]"
  - "[[concepts/pairing]]"
---

# Gateway プロトコル（解説）

> 原典: `raw/docs/gateway/protocol.md` ・ https://docs.openclaw.ai/ja-JP/gateway/protocol
>
> ℹ️ Gateway の **WS プロトコルの形式的な契約**（大）。[[concepts/architecture]] が「箱と線」の俯瞰、本ページがその**ワイヤー契約の詳細**。

## 一言まとめ

Gateway WS プロトコルは OpenClaw の**単一の制御プレーン＋ノードトランスポート**。すべてのクライアント（CLI・Web UI・macOS アプリ・iOS/Android/ヘッドレスノード）が WebSocket で接続し、ハンドシェイク時に**ロール＋スコープ**を宣言する。

## 位置づけ

[[concepts/architecture]] の WS トポロジーの「正確な契約」面。接続認証は [[concepts/authentication]]、デバイス信頼は [[concepts/pairing]]、運用視点のクイックリファレンスは [[sources/gateway/gateway]]。

## 仕組み・ふるまい

- **トランスポート**：WebSocket、テキストフレームに JSON。最初のフレームは必ず `connect`。
- **ハンドシェイク（connect）**：ロール（`operator`/`node`）＋スコープ＋認証＋デバイス ID＋`connect.challenge` nonce 署名（`v3` は platform/deviceFamily もバインド）。成功で **`hello-ok`**（presence/health/stateVersion/uptimeMs/制限・ポリシー、`features.methods`/`events` は保守的な検出リスト）。
- **フレーミング**：`req(method, params)` → `res(ok/payload|error)`、サーバープッシュ `event`。
- **ロール＋スコープ**：operator スコープ（`operator.read/write/admin/pairing/approvals/talk.secrets` → [[sources/gateway/operator-scopes]]）、node の caps/commands/permissions。
- **RPC メソッドファミリ**：chat/session/config/health/presence/agent/cron/tasks（タスク台帳）/models.list/Node ヘルパー/Operator ヘルパー等。**イベントファミリー**：`agent`/`chat`/`session.message`/`session.tool`/`sessions.changed`/`presence`/`tick`/`health`/`heartbeat`/`connect.challenge`/ペアリング/`shutdown`。
- **エージェント実行は 2 段階**：即時 ack（`status:"accepted"`）→ 最終 res（`ok|error`）、その間 `agent` イベントをストリーム。
- ブロードキャストイベントのスコープ設定、Node バックグラウンド生存イベント（`node.presence.alive`）、Exec 承認、Agent delivery fallback、TLS＋ピン留め、バージョニング（client constants）も規定。

## 設定・使い方の要点

- 認証は Gateway 認証モード（token/password/trusted-proxy/none → [[sources/gateway/configuration]]）。デバイス ID＋ペアリングは [[sources/gateway/pairing]]、移行診断あり。
- HTTP の別サーフェス（OpenAI 互換 `/v1/*`・`/tools/invoke`）は [[concepts/http-api]]。レガシー TCP ブリッジは削除（[[sources/gateway/bridge-protocol]]）。

## 注意点・落とし穴

- **イベントは再生されない**：シーケンスギャップ時は状態を取り直す。非 JSON/非 connect の最初のフレームは即クローズ。
- `hello-ok.features` は「呼べる全ルートのダンプ」ではなく検出用メタデータ。

## 用語と略称

- **制御プレーン（control plane）** = Gateway を操作・制御する面
- **ロール / スコープ** = 接続種別（operator/node）/ できることの範囲
- **RPC** = Remote Procedure Call（`req`/`res`）
- **nonce** = リプレイ防止の使い捨て乱数
- **hello-ok** = ハンドシェイク成功時のスナップショット応答

## 関連ページ

- [[concepts/architecture]] — WS トポロジーの俯瞰（本ページが詳細契約）
- [[components/gateway]] / [[concepts/pairing]] / [[concepts/authentication]]
- [[sources/gateway/operator-scopes]] / [[concepts/http-api]] / [[sources/gateway/bridge-protocol]]
