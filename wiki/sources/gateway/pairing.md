---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/pairing
source_path: raw/docs/gateway/pairing.md
doc_section: gateway
title: "Gateway 管理のペアリング"
ingested: 2026-06-14
tags: [pairing, node, device, trust, token, auto-approve]
related:
  - "[[concepts/pairing]]"
  - "[[components/node]]"
  - "[[concepts/security]]"
---

# Gateway 管理のペアリング（解説）

> 原典: `raw/docs/gateway/pairing.md` ・ https://docs.openclaw.ai/ja-JP/gateway/pairing

## 一言まとめ

ペアリングは「どのノードに参加を許すか」を **Gateway が信頼できる情報源**として決める仕組み。ノードは `connect` 時に**デバイスペアリング**（ロール `node`）を要求し、承認されると**認証トークン**が発行される（再ペアリングでローテーション）。UI/CLI は承認/拒否のフロントエンドにすぎない。

## 位置づけ

[[concepts/pairing]] の本体。[[components/node]] が Gateway に接続するときの信頼確立で、[[concepts/architecture]] の WS ハンドシェイクの一部。承認に必要な権限は [[sources/gateway/operator-scopes]]、接続認証（誰が Gateway につなげるか）とは別レイヤー（[[concepts/authentication]] の③デバイスペアリング）。

## 仕組み・ふるまい

1. ノードが Gateway WS に接続しペアリングを要求 → Gateway が**保留中リクエスト**を保存し `node.pair.requested` を発行。
2. CLI/UI で承認/拒否 → 承認時に**新トークン**を発行 → ノードがそのトークンで再接続して「ペアリング済み」に。**保留中は 5 分で期限切れ**。
3. ペアリング状態は `~/.openclaw/nodes/paired.json`/`pending.json`（`OPENCLAW_STATE_DIR` 配下、トークンはシークレット扱い）。

- **API**：イベント `node.pair.requested`/`resolved`、メソッド `node.pair.request`（ノードごと冪等）/`list`/`approve`（常に新トークン発行）/`reject`/`remove`/`verify`。承認に必要なスコープは宣言コマンドで決まる（コマンドなし=`operator.pairing`、非 exec=`+write`、`system.run`系=`+admin`）。
- ⚠️ **2026.3.31+ の破壊的変更**：ノードコマンドは**ノードペアリング承認まで無効**（デバイスペアリングだけでは不十分）。承認前にキューされたコマンドは破棄。ノード由来の実行は制限された信頼サーフェスに留まる（ホストレベルへ昇格できない）。

## 設定・使い方の要点

- CLI：`openclaw nodes pending|approve <id>|reject <id>|status|remove|rename`。`/pair qr` で QR ペアリングペイロードを生成。
- **自動承認**：①macOS アプリの**サイレント承認**（`silent` マーク＋同一ユーザーの SSH 検証）／②**信頼済み CIDR**（`gateway.nodes.pairing.autoApproveCidrs: ["192.168.1.0/24"]`、新規 `role: node` でスコープ要求なしのみ。LAN 全体の自動承認モードは無い）／③**メタデータアップグレード**（表示名等の非機密変更のみ。スコープ/公開鍵の昇格は対象外）。

## 注意点・落とし穴

- **ペアリングはトークン発行であって、ノードのコマンドサーフェスをノードごとに固定するものではない**。ライブコマンドは接続時の宣言＋グローバルポリシー（`gateway.nodes.allowCommands`/`denyCommands`）で決まり、`system.run` の許可は `exec.approvals.node.*`。
- ローカリティ：raw ソケットと上流プロキシ証拠の両方一致でのみループバック扱い。非ローカルを指す `X-Forwarded-*` があれば明示承認を要求（trusted-proxy 認証と同じルール）。
- `node.pair.*` は WS ハンドシェイクを制御する**デバイスペアリングとは別ストア**（明示的に `node.pair.*` を呼ぶクライアントのみ）。

## 用語と略称

- **ペアリング（pairing）** = ノード/デバイスの参加承認とトークン発行
- **保留中 / ペアリング済み** = 承認待ち / 承認済み（トークン保有）
- **CIDR** = ネットワーク範囲表記（`192.168.1.0/24`）
- **デバイストークン** = 承認時に発行される再接続用の秘密

## 関連ページ

- [[concepts/pairing]] — 対応する概念ページ
- [[components/node]] / [[concepts/authentication]] / [[concepts/security]]
- [[sources/gateway/operator-scopes]] / [[sources/gateway/protocol]] / [[sources/channels/pairing]]
