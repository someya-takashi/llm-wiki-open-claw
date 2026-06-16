---
type: component
aliases: [Node, ノード, role:node, ノードホスト, Node Host]
tags: [node, device, websocket, pairing, camera, location, system-run, voice]
concepts:
  - "[[concepts/architecture]]"
  - "[[concepts/pairing]]"
  - "[[concepts/discovery]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/voice]]"
  - "[[concepts/media-understanding]]"
related:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/architecture]]"
  - "[[sources/nodes/nodes]]"
  - "[[sources/gateway/pairing]]"
updated: 2026-06-14
---

# Node（ノード）

**Node** は、[[components/gateway]] と**同じ WS（WebSocket）サーバー**に `role: "node"` を宣言して接続する端末（macOS / iOS / Android / ヘッドレス）。コントロールプレーンの「クライアント」（アプリ・CLI・Web UI）と区別され、端末側の能力（カメラ・画面・位置・音声・OS コマンドなど）を `node.invoke` 経由で Gateway に**コマンドとして公開する**側に当たる。ノードは**周辺機器であって Gateway ではない**（Gateway サービスは実行しない）。位置づけは [[concepts/architecture]]。

## コマンドサーフェス

ノードは接続時に公開コマンドを宣言し、Gateway 側ポリシーが許可したものだけが使える。主なファミリー：

- **canvas.\***：WebView 制御（snapshot/present/eval、A2UI v0.8 JSONL プッシュ）。
- **camera.\* / screen.record**：写真（jpg）・動画クリップ（mp4・≤60s）・画面録画（[[sources/nodes/camera]]）。
- **location.get**：現在地（既定オフ、[[sources/nodes/location-command]]）。
- **talk.\* / voicewake**：オンデバイスの音声会話・ウェイクワード（[[concepts/voice]]）。
- **Android 個人データ**：`sms.send`・`device.*`・`notifications.*`・`photos.latest`・`contacts.*`・`calendar.*`・`callLog.*`・`motion.*`。
- **system.\***：`system.run`/`system.which`/`system.notify`（ノードホスト/Mac Node）。

⚠️ `canvas.*`/`camera.*`/`screen.*` は iOS/Android で**フォアグラウンド必須**（背景は `NODE_BACKGROUND_UNAVAILABLE`）。撮ったメディアは [[concepts/media-understanding]] で要約され得る。

## ノードホスト（リモート system.run）

Gateway があるマシンとは別のマシンでコマンドを実行したいとき、**ノードホスト**（`openclaw node run --host <gw> --port 18789`、UI なしのヘッドレス可）を使う。モデルは引き続き Gateway と話し、`host=node` の `exec` 呼び出しがノードホストへ転送される。loopback バインドの Gateway には SSH トンネル経由（[[concepts/remote-access]]）。

- 既定：`tools.exec.host=node` / `/exec host=node ...`、対象ノードは `tools.exec.node`（エージェントごと上書き可）。
- **Exec 承認はノードホストごと**（`~/.openclaw/exec-approvals.json`、`openclaw approvals allowlist add --node <id> "..."`）。承認は正規 `systemRunPlan` にバインドされ、承認後に command/cwd を編集すると拒否。
- ノードホストは `PATH` 上書きを無視し、危険な起動キー（`DYLD_*`/`LD_*`/`NODE_OPTIONS`/`PYTHON*` 等）を除去。

## ペアリングと信頼境界

ノードは `connect` に `role:"node"`・caps/commands/permissions・デバイス ID を含め、Gateway がデバイスペアリングを作成する（[[concepts/pairing]] / [[sources/gateway/pairing]]）。同一ホストの local loopback は自動承認できるが、**Tailnet/LAN・非ローカルは明示承認が必須**。すべての接続は `connect.challenge` の nonce に署名し、署名ペイロード `v3` は `platform`+`deviceFamily` もバインドする。Gateway を見つける手段（Bonjour/tailnet/SSH）は [[concepts/discovery]]。

⚠️ **3 つの別ゲート**を混同しない：①デバイスペアリング（接続できるか）②ノードコマンドポリシー（`gateway.nodes.allowCommands`/`denyCommands`＋プラットフォーム既定）③Exec 承認（シェルコマンドを実行できるか）。切り分けは [[sources/nodes/troubleshooting]]。ノード由来の実行は制限された信頼サーフェスに留まり、ホストレベルへ昇格できない（[[concepts/sandboxing]]・[[concepts/threat-model]]）。

## Mac Node モード

macOS メニューバーアプリは Node として Gateway WS に接続でき（`openclaw nodes …` がその Mac に効く）、リモートモードでは Gateway ポートへの SSH トンネルを開いて `localhost` に接続する。

## 関連

- [[concepts/architecture]] / [[concepts/pairing]] / [[concepts/discovery]] / [[concepts/sandboxing]]
- 能力：[[concepts/voice]] / [[concepts/media-understanding]] / [[concepts/remote-access]]
- [[sources/nodes/nodes]] — ノードの総覧 / [[sources/nodes/troubleshooting]] — 診断
- [[components/gateway]] / [[components/cli]]
