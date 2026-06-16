---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/web
source_path: raw/docs/web/web.md
doc_section: web
title: "ウェブ（Web サーフェス）"
ingested: 2026-06-14
tags: [web, control-ui, bind-mode, security, webhook, tailscale]
related:
  - "[[components/control-ui]]"
  - "[[concepts/remote-access]]"
  - "[[concepts/authentication]]"
---

# ウェブ（Web サーフェス）（解説）

> 原典: `raw/docs/web/web.md` ・ https://docs.openclaw.ai/ja-JP/web
>
> ℹ️ `web/` セクションのランディングページ。Gateway が出す Web サーフェス（Control UI・Webhook）の概観と、**バインドモード・セキュリティ**に焦点を当てる。

## 一言まとめ

Gateway は WS と**同じポート**から小さな**ブラウザー Control UI**（Vite + Lit）を配信する。本ページはその提供条件（既定 `http://<host>:18789/`・`basePath`）と、非 loopback 公開時のセキュリティ（認証必須・許可オリジン）をまとめる入口。

## 位置づけ

[[components/control-ui]] の提供面の概要（機能詳細は [[sources/web/control-ui]]）。外から開く手段は [[concepts/remote-access]]、認証は [[concepts/authentication]]。Webhook は同じ HTTP サーバー上に出る（`hooks.enabled=true`、設定は [[sources/gateway/configuration]] の `hooks`）。

## 仕組み・ふるまい

- Control UI はアセット（`dist/control-ui`）があれば**既定で有効**（`gateway.controlUi.enabled`/`basePath`）。
- Tailscale アクセス：①統合 Serve（loopback 維持＋ `tailscale: {mode:"serve"}`、`https://<magicdns>/`）／②tailnet バインド＋トークン（`bind:"tailnet"`＋`auth.token`、`http://<tailscale-ip>:18789/`）／③Funnel（公開、`auth.mode:"password"` 必須）。

## 設定・使い方の要点

- UI は `pnpm ui:build`（`dist/control-ui`）でビルド。
- 共有シークレットモードでは UI は `connect.params.auth.token`/`password` を送信。TLS 有効時はダッシュボードに `https://`・`wss://` を表示。

## 注意点・落とし穴

- ⚠️ **非 loopback バインドでも Gateway 認証は必須**（token/password/trusted-proxy/Tailscale Serve ID ヘッダー）。HTTP API（`/v1/*`）は Tailscale ID ヘッダーを使わず通常の HTTP 認証に従う。
- ⚠️ 非 loopback の Control UI は `gateway.controlUi.allowedOrigins`（完全オリジン）を明示しないと**起動が既定で拒否**。`dangerouslyAllowHostHeaderOriginFallback=true` は危険なセキュリティ低下。

## 用語と略称

- **Control UI** = Gateway が配信するブラウザー管理 UI
- **Vite + Lit** = フロントエンドのビルドツールと Web Components ライブラリ
- **bind モード** = Gateway の待受範囲（loopback/tailnet/lan/custom）
- **Webhook** = 外部から Gateway を叩く HTTP エンドポイント
- **allowedOrigins** = 接続を許す HTTP オリジンの許可リスト

## 関連ページ

- [[components/control-ui]] — ブラウザー管理 UI（詳細）
- [[components/webchat]] / [[components/cli]]
- [[concepts/remote-access]] / [[concepts/authentication]] / [[sources/gateway/tailscale]]
