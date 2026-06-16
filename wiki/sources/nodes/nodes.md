---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes
source_path: raw/docs/nodes/nodes.md
doc_section: nodes
title: "ノード"
ingested: 2026-06-14
tags: [node, device, commands, node-host, system-run, camera, location]
related:
  - "[[components/node]]"
  - "[[concepts/pairing]]"
  - "[[concepts/sandboxing]]"
---

# ノード（解説）

> 原典: `raw/docs/nodes/nodes.md` ・ https://docs.openclaw.ai/ja-JP/nodes
>
> ℹ️ `nodes/` セクションの**ランディングページ**。Node の全体像を概観し、camera/audio/location などの個別ページ（[[sources/nodes/camera]] 等）へ分岐する。

## 一言まとめ

ノード（Node, `role: "node"` で Gateway WS に接続するコンパニオンデバイス）が**何を公開し、どう承認され、どう呼び出されるか**の総覧。`node.invoke` 経由で `canvas.*`/`camera.*`/`device.*`/`system.*` などのコマンドサーフェスを出す「周辺機器」であり、**Gateway そのものではない**。

## 位置づけ

[[components/node]] の中核ソース。接続契約は [[sources/gateway/protocol]]、信頼確立は [[concepts/pairing]]、ツール実行の隔離・許可は [[concepts/sandboxing]]。レガシー TCP ブリッジ（[[sources/gateway/bridge-protocol]]）は履歴用途のみ。

## 仕組み・ふるまい

- **2 ゲートのコマンドポリシー**：①ノードが `connect.commands` でコマンドを宣言 → ②Gateway のプラットフォームポリシーが許可。安全なコマンド（`canvas.*`/`camera.list`/`location.get`/`screen.snapshot`、`talk` の push-to-talk）は既定許可、危険・プライバシー影響大（`camera.snap`/`camera.clip`/`screen.record`）は `gateway.nodes.allowCommands` で明示 opt-in。`gateway.nodes.denyCommands` が常に最優先。
- **メッセージは Gateway に届く**（Telegram/WhatsApp 等）。ノードには届かない。Gateway がエージェントを実行し、必要に応じて `node.*` RPC でノードを呼ぶ。
- **コマンドファミリー**：canvas（スナップショット/present/eval/A2UI v0.8 JSONL）、camera（snap=jpg/clip=mp4、フォアグラウンド必須・≤60s）、screen.record、location.get（既定オフ）、sms.send（Android）、device/notifications/photos/contacts/calendar/callLog/motion（Android 個人データ）、system（run/notify/which）。

## 設定・使い方の要点

- **ペアリング**：`openclaw devices list|approve|reject`、`openclaw nodes status|describe|rename`。承認スコープは宣言コマンド依存（なし=`operator.pairing`、非 exec=`+write`、`system.run`系=`+admin`）。詳細は [[sources/gateway/pairing]]。
- **リモートノードホスト（`system.run`）**：Gateway は別マシンに `exec` を転送できる。`openclaw node run --host <gw> --port 18789`（ヘッドレス。Linux/Windows 可）or `openclaw node install`。loopback バインドの Gateway には SSH トンネル経由（[[concepts/remote-access]]）。`exec host=node` の既定は `tools.exec.host=node` / `/exec host=node ...`、ノードバインドは `tools.exec.node`。
- **Exec 承認はノードホストごと**（`~/.openclaw/exec-approvals.json`）。`openclaw approvals allowlist add --node <id> "/usr/bin/uname"`。macOS Node モードはアプリの Settings → Exec approvals。

## 注意点・落とし穴

- ⚠️ `canvas.*`/`camera.*`/`screen.*` は iOS/Android で**フォアグラウンド必須**（背景は `NODE_BACKGROUND_UNAVAILABLE`）。
- ⚠️ **ペアリング ≠ コマンド承認 ≠ Exec 承認**：3 つの別ゲート（[[sources/nodes/troubleshooting]] の権限マトリクス）。`system.run`/`system.run.prepare` は `nodes invoke` には出さず exec パス専用。承認は正規 `systemRunPlan` にバインドされ、承認後に command/cwd を編集すると拒否。
- Node ホストは `PATH` 上書きを無視し危険な起動キー（`DYLD_*`/`LD_*`/`NODE_OPTIONS` 等）を除去。

## 用語と略称

- **ノード（Node）** = `role: node` で WS 接続するコンパニオンデバイス
- **node.invoke** = ノードコマンドを呼ぶ生 RPC
- **ノードホスト（node host）** = `system.run`/`system.which` を公開するヘッドレス Node
- **A2UI** = Agent-to-UI（エージェントが描画する UI、Canvas にプッシュ）
- **TCC** = macOS の権限管理（画面収録等の許可）

## 関連ページ

- [[components/node]] — 対応する構成要素ページ
- [[sources/nodes/camera]] / [[sources/nodes/audio]] / [[sources/nodes/location-command]] / [[sources/nodes/troubleshooting]]
- [[sources/nodes/media-understanding]] / [[sources/nodes/talk]] / [[concepts/pairing]] / [[concepts/remote-access]]
