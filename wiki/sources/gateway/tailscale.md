---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/tailscale
source_path: raw/docs/gateway/tailscale.md
doc_section: gateway
title: "Tailscale"
ingested: 2026-06-14
tags: [tailscale, serve, funnel, remote-access, bind-mode, identity-header]
related:
  - "[[concepts/remote-access]]"
  - "[[sources/gateway/remote]]"
  - "[[concepts/authentication]]"
---

# Tailscale（解説）

> 原典: `raw/docs/gateway/tailscale.md` ・ https://docs.openclaw.ai/ja-JP/gateway/tailscale

## 一言まとめ

Gateway の Control UI/WS 向けに Tailscale の **Serve（tailnet 専用）/Funnel（公開 HTTPS）を自動構成**する機能。Gateway は loopback に留まったまま、Tailscale が HTTPS・ルーティング・（Serve は）ID ヘッダーを提供する。

## 位置づけ

[[concepts/remote-access]] の Tailscale 経路（SSH トンネルの代替/補完、[[sources/gateway/remote]]）。Serve の ID ヘッダー認証は [[concepts/authentication]] の接続認証の一形態。

## 仕組み・ふるまい

- **モード**：`serve`（`tailscale serve`、Gateway は `127.0.0.1`）/`funnel`（`tailscale funnel`、公開 HTTPS・共有パスワード必須）/`off`（既定）。
- **Serve の ID 認証**：`tailscale.mode: "serve"`＋`gateway.auth.allowTailscale: true` で、トークン/パスワードなしに Tailscale ID ヘッダー（`tailscale-user-login`）で認証。Gateway は `tailscale whois` で `x-forwarded-for` を検証。⚠️ HTTP API（`/v1/*`・`/tools/invoke`）は**この ID 認証を使わず**通常の HTTP 認証モードに従う。
- **バインド代替**：`gateway.bind: "tailnet"` は tailnet IP 直接バインド（HTTPS/Serve なし、loopback は使えない）。

## 設定・使い方の要点

- 例：Serve = `{gateway:{bind:"loopback",tailscale:{mode:"serve"}}}`（`https://<magicdns>/`）。Funnel = `{tailscale:{mode:"funnel"},auth:{mode:"password"}}`。CLI `openclaw gateway --tailscale serve|funnel`。
- `resetOnExit`（終了時に serve/funnel を戻す）、`preserveFunnel`（再起動をまたいで Funnel ルート維持）。

## 注意点・落とし穴

- ⚠️ Funnel は認証モードが `password` でない限り起動拒否（公開露出回避）。Funnel の TLS 対応ポートは 443/8443/10000 のみ、v1.38.3+/MagicDNS/HTTPS 有効/funnel 属性が必要。
- `allowTailscale` のトークンなしフローは「Gateway ホストが信頼済み」前提。同ホストに信頼できないローカルコードがあるなら `false` にして token/password を使う。
- ブラウザー制御は Funnel を避け、オペレーターアクセス相当に扱う（tailnet 専用＋意図的ノードペアリング）。

## 用語と略称

- **Serve / Funnel** = Tailscale の tailnet 限定 / 公開インターネット向け HTTPS 公開
- **MagicDNS** = Tailscale のホスト名解決
- **ID ヘッダー（identity header）** = `tailscale-user-login` 等、認証済み ID を伝える HTTP ヘッダー

## 関連ページ

- [[concepts/remote-access]] / [[sources/gateway/remote]]
- [[concepts/authentication]] / [[concepts/discovery]] / [[sources/gateway/trusted-proxy-auth]]
