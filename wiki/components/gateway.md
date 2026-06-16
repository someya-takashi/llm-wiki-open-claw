---
type: component
aliases: [Gateway, ゲートウェイ, デーモン]
tags: [daemon, websocket, runtime, provider]
concepts:
  - "[[concepts/architecture]]"
  - "[[concepts/pairing]]"
  - "[[concepts/configuration]]"
related:
  - "[[components/node]]"
sources:
  - "[[sources/concepts/architecture]]"
  - "[[sources/gateway/gateway]]"
updated: 2026-06-14
---

# Gateway（ゲートウェイ）

**Gateway** は OpenClaw の中心となる **ホストごとにただ 1 つの常駐デーモン（daemon, 常時動くバックグラウンドプロセス）**。すべてのメッセージングプロバイダー接続を所有し、型付きの **WS（WebSocket）** API を公開して、クライアントと [[components/node]] からのリクエストに応えイベントを配信する。OpenClaw を「Gateway 中心のハブ＆スポーク」たらしめる主役で、全体像は [[concepts/architecture]] に対応する。

## 役割

- **プロバイダー接続の維持**：WhatsApp（Baileys）・Telegram（grammY）・Slack・Discord・Signal・iMessage・WebChat 等（[[concepts/channel-routing]]）。ホストごと 1 つなので、WhatsApp の単一セッションを開く唯一の場所。モデル供給元は [[concepts/model-providers]]（[[providers/anthropic]]/[[providers/openai]]/[[providers/google]]/[[providers/ollama]] 等）。
- **WS API の公開**：`{type:"req"...}` → `{type:"res"...}` と、サーバープッシュの `{type:"event"...}`。受信フレームを JSON Schema で検証し、`agent` `chat` `presence` `health` `heartbeat` `cron` などのイベントを発行。
- **エージェント実行の起点**：`agent` / `agent.wait` RPC が [[concepts/agent-loop]] を駆動する。
- **HTTP サーフェス**：`/__openclaw__/canvas/`（編集可能な HTML/CSS/JS）と `/__openclaw__/a2ui/`（A2UI ホスト）に加え、ブラウザー管理 UI [[components/control-ui]] を同一ポート（既定 `18789`）で配信。
- **クライアント面**：人間が話す主要 UI は [[components/control-ui]]（ブラウザー）・[[components/webchat]]（ネイティブチャット）・[[components/cli]]（端末 TUI）。いずれも同じ WS・認証・[[concepts/pairing]] を通る。
- **拡張**：チャネル・モデルプロバイダー・ツール・音声などは [[components/plugin-system]] が Gateway プロセス内に登録（多くの組み込み機能自体がコア Plugin）。外部 Plugin の配布は [[components/clawhub]]。

## 運用

- 起動：`openclaw gateway`（フォアグラウンド、ログは stdout）。**監視付き起動**は `openclaw gateway install`（macOS=launchd `ai.openclaw.gateway`／Linux=systemd／Windows=スケジュールタスク）。再起動は `openclaw gateway restart`（`stop`＋`start` を連結しない）。状態は `openclaw gateway status [--deep|--require-rpc]`。
- バインドは既定 `127.0.0.1:18789`（loopback）。ポート優先順位 `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789`。リモートは Tailscale/VPN 推奨、代替は SSH トンネル（認証はバイパスしない）。
- 認証（`gateway.auth.*`：`token`/`password`/`trusted-proxy`）は**ローカル/リモートを問わず全接続に適用**（→ [[concepts/authentication]] の「接続認証」）。新デバイスは [[concepts/pairing]] が必要。秘密値は [[concepts/secrets]]（SecretRef）で外部化。
- **設定**は [[concepts/configuration]]（`openclaw.json`、ホットリロード `hybrid` で `gateway.*` 変更時のみ再起動）。**OpenAI 互換 HTTP エンドポイント**（`/v1/chat/completions`・`/v1/responses`・`/tools/invoke` 等）は WS と同一ポートで多重化 → [[concepts/http-api]]。
- **検出・接続**：LAN/tailnet 上での発見と経路選択は [[concepts/discovery]]、外から安全に到達する手段（SSH/Tailscale）は [[concepts/remote-access]]、自前ホストの LLM を載せるなら [[concepts/local-models]]。
- **メディア・音声**：受信メディアの要約は [[concepts/media-understanding]]、聞く/話す音声機能は [[concepts/voice]]（ウェイクワードは Gateway が集約所有）。
- 詳細な起動・監視・失敗シグネチャは運用手順書 [[sources/gateway/gateway]]。

## 不変条件

ホストごと 1 つ・単一 Baileys セッション、ハンドシェイク必須、イベントは再生されない。詳細は [[sources/concepts/architecture]]。「ホストごと 1 つ」を強制するロック機構は [[sources/gateway/gateway-lock]]、意図的に複数を分離運用する方法（レスキューボット等）は [[sources/gateway/multiple-gateways]]。長時間コマンドの実行・管理は `exec`/`process` ツール（[[sources/gateway/background-process]]）。

## 関連

- [[concepts/architecture]] / [[concepts/agent-loop]] / [[concepts/pairing]] / [[concepts/configuration]] / [[concepts/authentication]]
- ネットワーク・API：[[concepts/discovery]] / [[concepts/remote-access]] / [[concepts/http-api]] / [[concepts/local-models]]
- メディア・音声：[[concepts/media-understanding]] / [[concepts/voice]]
- セキュリティ：[[concepts/security]] / [[concepts/sandboxing]] / [[concepts/secrets]] / [[concepts/threat-model]]
- チャネル・モデル：[[concepts/channel-routing]] / [[concepts/model-providers]] / [[concepts/model-failover]]
- 自動化：[[concepts/automation]] / [[concepts/cron]] / [[concepts/tasks]] / [[concepts/hooks]] / [[concepts/heartbeat]]
- 実行・拡張：[[concepts/exec]] / [[components/plugin-system]]
- 観測・運用：[[concepts/logging]] / [[concepts/observability]] / [[concepts/diagnostics]]
- クライアント：[[components/control-ui]] / [[components/webchat]] / [[components/cli]]
- [[sources/gateway/gateway]] — Gateway 運用手順書
- [[components/node]]
