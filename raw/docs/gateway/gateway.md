---
title: "Gateway 運用手順書"
source: "https://docs.openclaw.ai/ja-JP/gateway"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
このページは、Gatewayサービスの1日目の起動と2日目以降の運用に使用します。[**詳細なトラブルシューティング**

正確なコマンド手順とログシグネチャを使った、症状起点の診断。

](https://docs.openclaw.ai/ja-JP/gateway/troubleshooting)

[

**設定**

タスク指向のセットアップガイドと完全な設定リファレンス。

](https://docs.openclaw.ai/ja-JP/gateway/configuration)[

**シークレット管理**

SecretRef契約、ランタイムスナップショットの動作、移行/再読み込み操作。

](https://docs.openclaw.ai/ja-JP/gateway/secrets)[

**シークレット計画契約**

正確な `secrets apply` ターゲット/パスルールと、参照のみの認証プロファイル動作。

](https://docs.openclaw.ai/ja-JP/gateway/secrets-plan-contract)

## 5分のローカル起動

- ### Gatewayを起動する
	bash
	```bash
	openclaw gateway --port 18789
	# debug/trace mirrored to stdio
	openclaw gateway --port 18789 --verbose
	# force-kill listener on selected port, then start
	openclaw gateway --force
	```
- ### サービスの健全性を確認する
	bash
	```bash
	openclaw gateway status
	openclaw status
	openclaw logs --follow
	```
	健全なベースライン: `Runtime: running` 、 `Connectivity probe: ok` 、および期待内容に一致する `Capability: ...`。到達性だけでなく読み取りスコープのRPC証明が必要な場合は、 `openclaw gateway status --require-rpc` を使用します。
- ### チャネルの準備状態を検証する
	bash
	```bash
	openclaw channels status --probe
	```
	到達可能なGatewayがある場合、アカウントごとのライブチャネルプローブと任意の監査を実行します。 Gatewayに到達できない場合、CLIはライブプローブ出力ではなく、設定のみのチャネルサマリーにフォールバックします。

> [!note] Note
> **Note**
> 
> Gateway設定の再読み込みは、アクティブな設定ファイルパスを監視します（プロファイル/状態のデフォルトから解決されるか、設定されている場合は `OPENCLAW_CONFIG_PATH` から解決されます）。 デフォルトモードは `gateway.reload.mode="hybrid"` です。 最初の正常な読み込み後、実行中プロセスはアクティブなインメモリ設定スナップショットを提供します。正常な再読み込みでは、そのスナップショットがアトミックに差し替えられます。

## ランタイムモデル

- ルーティング、コントロールプレーン、チャネル接続のための常時稼働プロセス1つ。
- 次のための単一の多重化ポート:
	- WebSocketコントロール/RPC
		- HTTP API、OpenAI互換（ `/v1/models` 、 `/v1/embeddings` 、 `/v1/chat/completions` 、 `/v1/responses` 、 `/tools/invoke` ）
		- コントロールUIとフック
- デフォルトのバインドモード: `loopback` 。
- 認証はデフォルトで必須です。共有シークレットのセットアップでは `gateway.auth.token` / `gateway.auth.password` （または `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD` ）を使用し、非loopbackの リバースプロキシセットアップでは `gateway.auth.mode: "trusted-proxy"` を使用できます。

## OpenAI互換エンドポイント

OpenClawの最も効果の高い互換サーフェスは現在、次のとおりです。

- `GET /v1/models`
- `GET /v1/models/{id}`
- `POST /v1/embeddings`
- `POST /v1/chat/completions`
- `POST /v1/responses`

このセットが重要な理由:

- 多くのOpen WebUI、LobeChat、LibreChat統合は最初に `/v1/models` をプローブします。
- 多くのRAGとメモリパイプラインは `/v1/embeddings` を期待します。
- エージェントネイティブのクライアントは、 `/v1/responses` を好む傾向が強まっています。

計画メモ:

- `/v1/models` はエージェント優先です。 `openclaw` 、 `openclaw/default` 、 `openclaw/<agentId>` を返します。
- `openclaw/default` は、常に設定済みのデフォルトエージェントにマップされる安定したエイリアスです。
- バックエンドプロバイダー/モデルを上書きしたい場合は `x-openclaw-model` を使用します。それ以外の場合は、選択されたエージェントの通常のモデルと埋め込み設定が制御を維持します。

これらはすべてメインのGatewayポートで実行され、Gateway HTTP APIの他の部分と同じ信頼されたオペレーター認証境界を使用します。

### ポートとバインドの優先順位

| 設定 | 解決順序 |
| --- | --- |
| Gatewayポート | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| バインドモード | CLI/override → `gateway.bind` → `loopback` |

インストール済みGatewayサービスは、解決済みの `--port` をスーパーバイザーメタデータに記録します。 `gateway.port` を変更した後は、launchd/systemd/schtasksが新しいポートでプロセスを起動するように、 `openclaw doctor --fix` または `openclaw gateway install --force` を実行します。

Gateway起動では、非loopbackバインド用のローカル コントロールUIオリジンをシードするときに、同じ有効ポートとバインドを使用します。たとえば、 `--bind lan --port 3000` はランタイム 検証の実行前に `http://localhost:3000` と `http://127.0.0.1:3000` をシードします。HTTPSプロキシURLなどのリモートブラウザーオリジンは、 `gateway.controlUi.allowedOrigins` に明示的に追加してください。

### ホットリロードモード

| `gateway.reload.mode` | 動作 |
| --- | --- |
| `off` | 設定を再読み込みしない |
| `hot` | ホットセーフな変更のみ適用 |
| `restart` | 再起動が必要な変更で再起動する |
| `hybrid` （デフォルト） | 安全な場合はホット適用し、必要な場合は再起動する |

## オペレーターコマンドセット

bash

```bash
openclaw gateway status
openclaw gateway status --deep   # adds a system-level service scan
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

`gateway status --deep` は追加のサービス検出（LaunchDaemons/systemdシステム ユニット/schtasks）のためのものであり、より深いRPC健全性プローブではありません。

## 複数のGateway（同一ホスト）

ほとんどのインストールでは、1台のマシンにつき1つのGatewayを実行するべきです。単一のGatewayで複数の エージェントとチャネルをホストできます。

意図的に分離やレスキューボットが必要な場合にのみ、複数のGatewayが必要です。

有用な確認:

bash

```bash
openclaw gateway status --deep
openclaw gateway probe
```

想定される内容:

- `gateway status --deep` は、古いlaunchd/systemd/schtasksインストールがまだ残っている場合に `Other gateway-like services detected (best effort)` を報告し、クリーンアップのヒントを出力できます。
- 複数のターゲットが応答する場合、 `gateway probe` は `multiple reachable gateways` について警告できます。
- それが意図したものである場合は、Gatewayごとにポート、設定/状態、ワークスペースルートを分離してください。

インスタンスごとのチェックリスト:

- 一意の `gateway.port`
- 一意の `OPENCLAW_CONFIG_PATH`
- 一意の `OPENCLAW_STATE_DIR`
- 一意の `agents.defaults.workspace`

例:

bash

```bash
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

詳細なセットアップ: [/gateway/multiple-gateways](https://docs.openclaw.ai/ja-JP/gateway/multiple-gateways) 。

## リモートアクセス

推奨: Tailscale/VPN。 フォールバック: SSHトンネル。

bash

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

次に、クライアントをローカルで `ws://127.0.0.1:18789` に接続します。

> [!note] Note
> **Warning**
> 
> SSHトンネルはGateway認証をバイパスしません。共有シークレット認証では、トンネル経由であってもクライアントは `token` / `password` を送信する必要があります。IDを含むモードでは、 リクエストは引き続きその認証パスを満たす必要があります。

参照: [リモートGateway](https://docs.openclaw.ai/ja-JP/gateway/remote) 、 [認証](https://docs.openclaw.ai/ja-JP/gateway/authentication) 、 [Tailscale](https://docs.openclaw.ai/ja-JP/gateway/tailscale) 。

## 監視とサービスライフサイクル

本番環境に近い信頼性のために、監視付き実行を使用します。

### macOS（launchd）

bash

```bash
openclaw gateway install
openclaw gateway status
openclaw gateway restart
openclaw gateway stop
```

再起動には `openclaw gateway restart` を使用します。再起動の代用として `openclaw gateway stop` と `openclaw gateway start` を連結しないでください。

macOSでは、 `gateway stop` はデフォルトで `launchctl bootout` を使用します。これは無効化を永続化せずに現在のブートセッションからLaunchAgentを削除するため、予期しないクラッシュ後もKeepAlive自動復旧が機能し、 `gateway start` できれいに再有効化されます。再起動をまたいで自動再生成を永続的に抑制するには、 `--disable` を渡します: `openclaw gateway stop --disable` 。

LaunchAgentラベルは `ai.openclaw.gateway` （デフォルト）または `ai.openclaw.<profile>` （名前付きプロファイル）です。 `openclaw doctor` はサービス設定のドリフトを監査し、修復します。

### Linux（systemdユーザー）

bash

```bash
openclaw gateway install
systemctl --user enable --now openclaw-gateway[-<profile>].service
openclaw gateway status
```

ログアウト後も永続化するには、lingeringを有効にします。

bash

```bash
sudo loginctl enable-linger <user>
```

カスタムインストールパスが必要な場合の手動ユーザーユニット例:

ini

```
[Unit]
Description=OpenClaw Gateway
After=network-online.target
Wants=network-online.target
 
[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5
TimeoutStopSec=30
TimeoutStartSec=30
SuccessExitStatus=0 143
KillMode=control-group
 
[Install]
WantedBy=default.target
```

### Windows（ネイティブ）

powershell

```powershell
openclaw gateway install
openclaw gateway status --json
openclaw gateway restart
openclaw gateway stop
```

ネイティブWindowsの管理付き起動では、 `OpenClaw Gateway` という名前のスケジュールされたタスク （名前付きプロファイルでは `OpenClaw Gateway (<profile>)` ）を使用します。スケジュールされたタスクの 作成が拒否された場合、OpenClawは状態ディレクトリ内の `gateway.cmd` を指すユーザーごとのStartupフォルダーランチャーにフォールバックします。

### Linux（システムサービス）

マルチユーザー/常時稼働ホストではシステムユニットを使用します。

bash

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now openclaw-gateway[-<profile>].service
```

ユーザーユニットと同じサービス本文を使用しますが、 `/etc/systemd/system/openclaw-gateway[-<profile>].service` の下にインストールし、 `openclaw` バイナリが別の場所にある場合は `ExecStart=` を調整してください。

同じプロファイル/ポートに対して、 `openclaw doctor --fix` にもユーザーレベルのGatewayサービスをインストールさせないでください。DoctorはシステムレベルのOpenClaw Gatewayサービスを検出すると、その自動インストールを拒否します。システムユニットがライフサイクルを所有する場合は、 `OPENCLAW_SERVICE_REPAIR_POLICY=external` を使用します。

## 開発プロファイルのクイックパス

bash

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

デフォルトには、分離された状態/設定とベースGatewayポート `19001` が含まれます。

## プロトコルクイックリファレンス（オペレーター視点）

- 最初のクライアントフレームは `connect` である必要があります。
- Gatewayは `hello-ok` スナップショット（ `presence` 、 `health` 、 `stateVersion` 、 `uptimeMs` 、制限/ポリシー）を返します。
- `hello-ok.features.methods` / `events` は保守的な検出リストであり、 呼び出し可能なすべてのヘルパールートを生成してダンプしたものではありません。
- リクエスト: `req(method, params)` → `res(ok/payload|error)` 。
- 一般的なイベントには、 `connect.challenge` 、 `agent` 、 `chat` 、 `session.message` 、 `session.tool` 、 `sessions.changed` 、 `presence` 、 `tick` 、 `health` 、 `heartbeat` 、ペアリング/承認ライフサイクルイベント、 `shutdown` が含まれます。

エージェント実行は2段階です。

1. 即時の受理確認（ `status:"accepted"` ）
2. 最終完了レスポンス（ `status:"ok"|"error"` ）。その間に `agent` イベントがストリーミングされます。

完全なプロトコルドキュメントを参照: [Gatewayプロトコル](https://docs.openclaw.ai/ja-JP/gateway/protocol) 。

## 運用チェック

### Liveness

- WSを開いて `connect` を送信します。
- スナップショットを含む `hello-ok` レスポンスを期待します。

### Readiness

bash

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### ギャップ復旧

イベントは再生されません。シーケンスギャップが発生した場合は、続行する前に状態（ `health` 、 `system-presence` ）を更新します。

## 一般的な失敗シグネチャ

| シグネチャ | 想定される問題 |
| --- | --- |
| `refusing to bind gateway ... without auth` | 有効な Gateway 認証パスなしで非ループバックにバインドしている |
| `another gateway instance is already listening` / `EADDRINUSE` | ポートの競合 |
| `Gateway start blocked: set gateway.mode=local` | 設定がリモートモードに設定されているか、破損した設定からローカルモードのスタンプが欠落している |
| `unauthorized` during connect | クライアントと Gateway の間の認証不一致 |

完全な診断手順は、 [Gateway のトラブルシューティング](https://docs.openclaw.ai/ja-JP/gateway/troubleshooting) を使用してください。

## 安全性の保証

- Gateway プロトコルクライアントは、Gateway が利用できない場合に即座に失敗します（暗黙的な直接チャネルフォールバックはありません）。
- 無効な、または接続ではない最初のフレームは拒否され、閉じられます。
- 正常なシャットダウンでは、ソケットを閉じる前に `shutdown` イベントが発行されます。

---

関連:

- [トラブルシューティング](https://docs.openclaw.ai/ja-JP/gateway/troubleshooting)
- [バックグラウンドプロセス](https://docs.openclaw.ai/ja-JP/gateway/background-process)
- [設定](https://docs.openclaw.ai/ja-JP/gateway/configuration)
- [健全性](https://docs.openclaw.ai/ja-JP/gateway/health)
- [Doctor](https://docs.openclaw.ai/ja-JP/gateway/doctor)
- [認証](https://docs.openclaw.ai/ja-JP/gateway/authentication)

## 関連

- [設定](https://docs.openclaw.ai/ja-JP/gateway/configuration)
- [Gateway のトラブルシューティング](https://docs.openclaw.ai/ja-JP/gateway/troubleshooting)
- [リモートアクセス](https://docs.openclaw.ai/ja-JP/gateway/remote)
- [シークレット管理](https://docs.openclaw.ai/ja-JP/gateway/secrets)