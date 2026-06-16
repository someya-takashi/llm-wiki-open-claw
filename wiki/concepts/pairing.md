---
type: concept
aliases: [Pairing, デバイスペアリング, ノードペアリング]
tags: [pairing, node, device, trust, authentication]
related:
  - "[[concepts/authentication]]"
  - "[[concepts/architecture]]"
  - "[[concepts/discovery]]"
  - "[[concepts/security]]"
components:
  - "[[components/node]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/pairing]]"
  - "[[sources/channels/pairing]]"
  - "[[sources/gateway/discovery]]"
  - "[[sources/network]]"
updated: 2026-06-14
---

# ペアリング（Pairing）

ペアリング（pairing, 明示的なアクセス承認ステップ）は、「**誰/どのデバイスに参加を許すか**」を [[components/gateway]] が**唯一の信頼できる情報源**として決める仕組み。2 か所で使われる——**DM ペアリング**（ボットと会話できる人の承認、[[concepts/channel-routing]] 側）と**Node ペアリング**（Gateway に繋げるデバイス/Node の承認）。UI（macOS アプリ）や CLI は承認/拒否のフロントエンドにすぎず、判断はすべて Gateway が所有する。両者の入口は [[sources/channels/pairing]]、Node 側の詳細は [[sources/gateway/pairing]]。

## 認証 3 レイヤーの中での位置

[[concepts/authentication]] が整理するとおり、OpenClaw の信頼は 3 層に分かれる。ペアリングはその **③デバイス/ノード ID 層**にあたる：

| 層 | 何を守るか | 仕組み |
|---|---|---|
| ① 接続認証 | 誰が Gateway につなげるか | token / password / trusted-proxy / none |
| ② モデル認証 | どのプロバイダー資格情報を使うか | [[concepts/oauth]] / API キー |
| ③ **デバイスペアリング** | **どのノードを信頼するか** | **本ページ** |

①が「Gateway の入口の鍵」なら、③は「つないできた個々のノードを信頼済みとして登録し、再接続用トークンを与える」段階。両者は別レイヤーで、片方を満たしても他方は自動では満たされない。

## 仕組み（要点）

1. ノードが [[concepts/architecture]] の WS（WebSocket）ハンドシェイクで接続し、**デバイスペアリング**（ロール `node`）を要求。
2. Gateway が**保留中リクエスト**を保存し `node.pair.requested` を発行（保留は **5 分**で期限切れ）。
3. オペレーターが CLI/UI で承認 → Gateway が**新トークン**を発行（再ペアリングでローテーション）。
4. ノードがそのトークンで再接続し「ペアリング済み」に。状態は `~/.openclaw/nodes/paired.json`/`pending.json`。

詳細な API（`node.pair.request`/`approve`/`reject`/`remove`/`verify`）・CLI（`openclaw nodes pending|approve|reject`）・自動承認（macOS サイレント承認・信頼済み CIDR・メタデータアップグレード）は [[sources/gateway/pairing]] を参照。

## 既存 wiki とのつながり

ペアリングは「[[concepts/discovery]] で Gateway を**見つけた**後、実際に**信頼を確立する**」段階に位置する——検出ヒント（Bonjour の TXT 等）は認証されないので、最終的な信頼判断はペアリングが担う。承認に必要な権限は [[sources/gateway/operator-scopes]] のオペレータースコープ（コマンドなし=`operator.pairing`、`system.run` 系=`+operator.admin`）で段階化され、これは [[concepts/security]] の最小権限思想の一部。

⚠️ **重要な誤解**：ペアリングは「トークン発行と ID 確立」であって、ノードのコマンドサーフェスをノードごとに固定するものでは**ない**。ライブコマンドは接続時の宣言＋グローバルポリシー（`gateway.nodes.allowCommands`/`denyCommands`）と `exec.approvals.node.*` で決まる。さらに 2026.3.31+ では**ノードコマンドはノードペアリング承認まで無効**（デバイスペアリングだけでは不十分）。

## 代表ソース

- [[sources/gateway/pairing]] — Gateway 管理のペアリングの詳細（本概念の中核）
- [[sources/gateway/discovery]] — 見つける→つなぐ→ペアリングの流れ
- [[sources/network]] — localhost/LAN/tailnet での接続・ペアリング・保護のハブ

## 関連ページ

- [[concepts/authentication]] / [[concepts/architecture]] / [[concepts/discovery]] / [[concepts/security]]
- [[components/node]] / [[components/gateway]]
