---
type: concept
aliases: [Architecture, Gateway Architecture, Gateway アーキテクチャ]
tags: [gateway, websocket, runtime, pairing]
related:
  - "[[concepts/agent-loop]]"
  - "[[concepts/pairing]]"
  - "[[concepts/discovery]]"
  - "[[concepts/http-api]]"
components:
  - "[[components/gateway]]"
  - "[[components/node]]"
sources:
  - "[[sources/concepts/architecture]]"
  - "[[sources/gateway/protocol]]"
updated: 2026-06-14
---

# アーキテクチャ（Gateway 中心の WS トポロジー）

OpenClaw の全体像は **ハブ＆スポーク**で捉えられる。中心（ハブ）に **ホストごと 1 つの常駐デーモン [[components/gateway]]（Gateway）** があり、そこから伸びるスポークが、各メッセージングプロバイダー（WhatsApp/Telegram/Slack/…）と、Gateway に **WS（WebSocket, 双方向通信トランスポート）** でつながる 2 種類の接続先――**クライアント**（[[components/control-ui]]＝ブラウザー管理 UI・[[components/webchat]]＝ネイティブチャット・[[components/cli]]＝端末 TUI）と **[[components/node]]（`role: node` の端末）**――である。

## なぜこの形なのか

「メッセージング接続を所有する場所をホストに 1 つだけ」に絞ることで、WhatsApp の単一 Baileys セッションのような**重複できない接続を一意に保ち**、状態（presence・health）と認証・[[concepts/pairing]] を一箇所で扱える。クライアントや端末はステートレスに付け外しでき、Gateway 側が真実の源（source of truth）になる。これは「自分のインフラ上で完結させる」OpenClaw のセルフホスト思想とも噛み合う。

## 押さえる点

- **WS ワイヤープロトコル**：最初のフレームは必ず `connect`、以降は `req`/`res` と サーバープッシュの `event`。型は TypeBox → JSON Schema → Swift モデルへ生成。形式的な契約の全体は [[sources/gateway/protocol]]。
- **接続まわり**：Gateway を見つける手段（Bonjour/tailnet/SSH）は [[concepts/discovery]]、OpenAI 互換の HTTP サーフェスは [[concepts/http-api]]。
- **不変条件**：Gateway はホストごと 1 つ・単一 Baileys セッション、ハンドシェイク必須、**イベントは再生されない**（取りこぼしは取り直す）。
- **境界（trust）**：local loopback は自動承認可、Tailnet/LAN・非ローカルは明示ペアリング必須。Gateway 認証は全接続に適用。

詳細・接続ライフサイクル図・リモートアクセス手段は解説ページ [[sources/concepts/architecture]] を参照。

## 代表的な構成要素

- [[components/gateway]] — 中心デーモン
- [[components/node]] — `role: node` の端末
- クライアント UI：[[components/control-ui]] / [[components/webchat]] / [[components/cli]]

## 関連

- [[concepts/agent-loop]] — Gateway の `agent` RPC から始まる実行サイクル
- [[concepts/pairing]] — デバイス承認とトークン
- [[concepts/discovery]] — Gateway の発見と接続経路 / [[concepts/http-api]] — OpenAI 互換 HTTP
- [[overview]]
