---
type: concept
aliases: [Remote Access, リモートアクセス, SSH トンネル, Tailscale]
tags: [remote-access, ssh-tunnel, tailscale, bind-mode, vpn, credentials]
related:
  - "[[concepts/discovery]]"
  - "[[concepts/authentication]]"
  - "[[concepts/security]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/remote]]"
  - "[[sources/gateway/tailscale]]"
  - "[[sources/gateway/remote-gateway-readme]]"
updated: 2026-06-14
---

# リモートアクセス（Remote Access）

リモートアクセス（remote access, 別の場所から Gateway に安全に到達すること）は、専用ホスト（VPS/ホームサーバー/デスクトップ）で動く単一の Gateway（マスター）へ、ラップトップやモバイルから接続するための構成パターン群。一貫した原則は「**確信が無ければ Gateway は loopback 専用に保ち、外からの到達は SSH か Tailscale で被せる**」。

## なぜ重要か

Gateway は状態・認証プロファイル・チャネル・WhatsApp の単一セッションを所有する（[[concepts/architecture]]）。だからエージェントは「常時稼働の 1 ホスト」に置き、スリープしがちなラップトップはそこへ**接続するだけ**にしたい。リモートアクセスはその「置き場所と到達手段」を、公開露出を増やさずに解く。

## 2 つの到達手段

| 手段 | 内容 | 向き |
|---|---|---|
| **SSH トンネル** | `ssh -N -L 18789:127.0.0.1:18789` で loopback ポートを転送 | 汎用フォールバック。SSH が通ればどこでも、新規ポート不要 |
| **Tailscale** | Serve（tailnet 専用 HTTPS）/ Funnel（公開 HTTPS） | tailnet 内で最良の UX、HTTPS と ID ヘッダーを Tailscale が提供 |

bind モードは `loopback`（既定）/`tailnet`（tailnet IP 直接）/`lan`/`custom`/`auto`。非 loopback バインドは必ず Gateway 認証（[[concepts/authentication]]）を伴う。これは [[concepts/discovery]] の「Direct WS か SSH フォールバック」の運用面にあたる。

## 仕組み（要点）

- **macOS 永続トンネル**：`~/.ssh/config` の `LocalForward` ＋ LaunchAgent（`KeepAlive`/`RunAtLoad`）で再起動・クラッシュをまたぐ（[[sources/gateway/remote-gateway-readme]] のステップ手順）。
- **Tailscale Serve の ID 認証**：`gateway.auth.allowTailscale: true` でトークン/パスワードなしに `tailscale-user-login` ヘッダーで認証（`tailscale whois` 検証）。⚠️ ただし HTTP API（`/v1/*`・`/tools/invoke`、[[concepts/http-api]]）はこの ID 認証を使わない。
- **認証情報の優先順位**：ローカル/リモートモードで `gateway.auth.*` と `gateway.remote.*` の解決順が異なる。`--url` 上書き時は暗黙の config/env 認証にフォールバックしない。詳細は [[sources/gateway/remote]]。

## 既存 wiki とのつながり

⚠️ **loopback + SSH/Tailscale Serve が最も安全な既定**（公開露出ゼロ）。平文 `ws://` は既定 loopback 専用で、信頼済み private 網のみ `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` の緊急回避。Funnel は `password` 認証でないと起動を拒否し、ブラウザー制御はオペレーターアクセス相当（tailnet 専用＋意図的 [[concepts/pairing]]）に扱う——いずれも [[concepts/security]] と [[concepts/threat-model]]（公開 Gateway の検出リスク）の帰結。

## 代表ソース

- [[sources/gateway/remote]] — リモートアクセスの中核（SSH トンネル・bind・認証情報優先順位）
- [[sources/gateway/tailscale]] — Serve/Funnel と ID ヘッダー認証
- [[sources/gateway/remote-gateway-readme]] — OpenClaw.app の SSH トンネル手順（統合済み）

## 関連ページ

- [[concepts/discovery]] / [[concepts/authentication]] / [[concepts/security]]
- [[components/gateway]] / [[concepts/architecture]]
