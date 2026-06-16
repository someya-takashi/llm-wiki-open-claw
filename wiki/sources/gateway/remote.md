---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/remote
source_path: raw/docs/gateway/remote.md
doc_section: gateway
title: "リモートアクセス"
ingested: 2026-06-14
tags: [remote-access, ssh-tunnel, tailnet, bind-mode, credentials, launchagent]
related:
  - "[[concepts/remote-access]]"
  - "[[components/gateway]]"
  - "[[concepts/authentication]]"
---

# リモートアクセス（解説）

> 原典: `raw/docs/gateway/remote.md` ・ https://docs.openclaw.ai/ja-JP/gateway/remote

## 一言まとめ

専用ホスト（デスクトップ/サーバー）で単一の Gateway（マスター）を動かし、**SSH トンネルや Tailscale でそこへ接続**する構成。中心思想は「**確信が無ければ Gateway は loopback 専用に保つ**」。

## 位置づけ

[[concepts/remote-access]] の中核ソース。loopback バインド（[[sources/network]]）を維持したまま外から届かせる方法で、認証は [[concepts/authentication]]、Tailscale 詳細は [[sources/gateway/tailscale]]。

## 仕組み・ふるまい

- **中心モデル**：Gateway WS は既定 `127.0.0.1:18789`（loopback）。リモートはその loopback ポートを SSH 転送するか、tailnet/VPN でトンネルを減らす。
- **構成パターン**：①tailnet 内の常時稼働 Gateway（VPS/ホームサーバー＋ Tailscale Serve、ラップトップがスリープしてもエージェント常時稼働）／②ホームデスクトップが Gateway（macOS アプリの Remote over SSH）／③ラップトップが Gateway（他マシンから SSH or Control UI を Tailscale Serve）。
- **コマンドフロー**：メッセージ→Gateway→エージェント→`node.*` RPC でノード呼び出し→返信。ノードは Gateway サービスを実行しない（周辺機器）。
- **SSH トンネル**：`ssh -N -L 18789:127.0.0.1:18789 user@host`。起動中は `openclaw health`/`status --deep` が `ws://127.0.0.1:18789` 経由でリモートに届く。

## 設定・使い方の要点

- CLI リモート既定：`gateway.mode: "remote"`＋`gateway.remote.url`/`token`。`--url` を渡すと config/env 認証にフォールバックしない（`--token`/`--password` 必須）。
- **認証情報の優先順位**：明示（`--token` 等）が最優先。ローカルモード token は `OPENCLAW_GATEWAY_TOKEN`→`gateway.auth.token`→`gateway.remote.token`、リモートモードは `gateway.remote.token`→`OPENCLAW_GATEWAY_TOKEN`→`gateway.auth.token`。Node ホストのローカルモードは `gateway.remote.*` を無視。
- **macOS 永続 SSH トンネル**：`~/.ssh/config` の `LocalForward 18789 127.0.0.1:18789` ＋ LaunchAgent `ai.openclaw.ssh-tunnel.plist`（`KeepAlive`/`RunAtLoad`）。

## 注意点・落とし穴

- ⚠️ **loopback + SSH/Tailscale Serve が最も安全な既定**（公開露出なし）。平文 `ws://` は既定 loopback 専用（信頼済み private 網のみ緊急回避 `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1`）。
- 非 loopback バインド（lan/tailnet/custom）は Gateway 認証必須（token/password/trusted-proxy）。`gateway.remote.token` はクライアント側の資格情報でサーバー認証ではない。`wss://` は `gateway.remote.tlsFingerprint` でピン留め。

## 用語と略称

- **SSH トンネル** = SSH の `-L` 転送で loopback ポートを安全に中継
- **tailnet / Tailscale Serve** = Tailscale の仮想 VPN / tailnet 内 HTTPS 公開
- **LaunchAgent** = macOS のログイン時自動起動の仕組み
- **マスター Gateway** = 状態/チャネルを所有する単一の Gateway

## 関連ページ

- [[concepts/remote-access]] — 対応する概念ページ
- [[sources/gateway/tailscale]] / [[sources/gateway/remote-gateway-readme]]
- [[concepts/authentication]] / [[concepts/discovery]] / [[components/gateway]]
