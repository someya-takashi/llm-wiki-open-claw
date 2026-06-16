---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway
source_path: raw/docs/gateway/gateway.md
doc_section: gateway
title: "Gateway 運用手順書"
ingested: 2026-06-14
tags: [gateway, operations, runbook, service, ports, openai-compat]
related:
  - "[[components/gateway]]"
  - "[[concepts/architecture]]"
  - "[[concepts/configuration]]"
---

# Gateway 運用手順書（解説）

> 原典: `raw/docs/gateway/gateway.md` ・ https://docs.openclaw.ai/ja-JP/gateway
>
> ℹ️ これは `gateway/` セクションのランディングページ（URL は `/ja-JP/gateway`）。Gateway サービスの**起動と日々の運用**の手順書。

## 一言まとめ

[[components/gateway]] を**サービスとして起動・監視・運用**するための実務ガイド――5 分のローカル起動、オペレーターコマンド、launchd/systemd/Windows での監視付き起動、複数 Gateway、リモートアクセス、失敗シグネチャまで。

## 位置づけ

[[concepts/architecture]] が「Gateway とは何か」を、[[components/gateway]] が構成要素としての俯瞰を述べるのに対し、本ページは**運用面（1 日目の起動と 2 日目以降の運用）**。設定の中身は [[concepts/configuration]]、トラブルシュートは gateway/troubleshooting。

## 仕組み・ふるまい

### ランタイムモデル

- ルーティング・コントロールプレーン・チャネル接続のための**常時稼働プロセス 1 つ**。
- **単一の多重化ポート**で WS コントロール/RPC・HTTP API（OpenAI 互換）・Control UI/フックを提供。既定バインドは `loopback`、**認証は既定で必須**（共有シークレット `gateway.auth.token`/`password`、または非ループバックのリバースプロキシでは `gateway.auth.mode: "trusted-proxy"`）。
- **OpenAI 互換エンドポイント**（同じ Gateway ポート上）：`GET /v1/models`・`/v1/embeddings`・`/v1/chat/completions`・`/v1/responses`・`/tools/invoke`。`/v1/models` はエージェント優先で `openclaw`/`openclaw/default`/`openclaw/<agentId>` を返す。バックエンド上書きは `x-openclaw-model` ヘッダ。

### ポート/バインドの優先順位

- ポート：`--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789`。バインド：CLI/override → `gateway.bind` → `loopback`。
- `gateway.port` を変えたら `openclaw doctor --fix` か `openclaw gateway install --force` でスーパーバイザー（launchd/systemd/schtasks）を新ポートに合わせる。

### ホットリロード

`gateway.reload.mode`：`off`/`hot`（ホットセーフのみ）/`restart`/`hybrid`（既定。安全ならホット適用、必要なら再起動）。詳細は [[concepts/configuration]]。

## 設定・使い方の要点

- **5 分起動**：`openclaw gateway --port 18789`（`--verbose`/`--force`）。健全性 `openclaw gateway status`・`openclaw status`・`openclaw logs --follow`（`Runtime: running`・`Connectivity probe: ok`）。チャネル検証 `openclaw channels status --probe`。
- **オペレーターコマンド**：`gateway status [--deep|--json|--require-rpc]`・`gateway install/restart/stop`・`secrets reload`・`doctor`。
- **監視付き起動**：macOS は `openclaw gateway install`（launchd、ラベル `ai.openclaw.gateway`、`stop` は既定 `launchctl bootout`＝KeepAlive 復旧を残す、永続無効化は `--disable`）。Linux は systemd ユーザーユニット（`enable-linger` で永続化）またはシステムユニット。Windows はスケジュールタスク。
- **再起動は `openclaw gateway restart`**（`stop`＋`start` を連結しない）。
- **複数 Gateway（同一ホスト）**：基本は 1 台 1 Gateway。意図的な分離が要る場合のみ、各インスタンスに一意の `gateway.port`・`OPENCLAW_CONFIG_PATH`・`OPENCLAW_STATE_DIR`・`agents.defaults.workspace`。
- **リモート**：Tailscale/VPN 推奨、フォールバックは SSH トンネル（`ssh -N -L 18789:127.0.0.1:18789 user@host`）。**トンネルは認証をバイパスしない**（`token`/`password` は依然必要）。

## 注意点・落とし穴

- **一般的な失敗シグネチャ**：`refusing to bind gateway ... without auth`（非ループバックに認証なし）・`EADDRINUSE`/`another gateway instance is already listening`（ポート競合）・`Gateway start blocked: set gateway.mode=local`（設定がリモートモード/壊れた設定）・`unauthorized`（認証不一致）。
- **安全性の保証**：Gateway 不在時に暗黙の直接チャネルフォールバックはせず即失敗。最初のフレームが非 connect なら拒否してクローズ。正常終了時はクローズ前に `shutdown` イベント。
- **イベントは再生されない**：シーケンスギャップが出たら `health`/`system-presence` で状態を取り直す（[[concepts/presence]]）。

## 用語と略称

- **runbook（運用手順書）** = 起動・監視・障害対応の手順集
- **OpenAI 互換エンドポイント** = `/v1/*` で OpenAI API 形式を受ける HTTP サーフェス
- **launchd / systemd / schtasks** = macOS / Linux / Windows のサービス監視機構
- **liveness / readiness** = 生存確認 / 受付可能確認
- **KeepAlive** = プロセス異常終了時の自動復旧（launchd）

## 関連ページ

- [[components/gateway]] — 構成要素としての Gateway
- [[concepts/architecture]] / [[concepts/configuration]] / [[concepts/presence]]
- [[sources/gateway/configuration]] / [[sources/gateway/configuration-reference]]
