---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/web/control-ui
source_path: raw/docs/web/control-ui.md
doc_section: web
title: "Control UI（制御 UI）"
ingested: 2026-06-14
tags: [control-ui, dashboard, browser, spa, pairing, pwa, config]
related:
  - "[[components/control-ui]]"
  - "[[concepts/pairing]]"
  - "[[concepts/configuration]]"
---

# Control UI（制御 UI）（解説）

> 原典: `raw/docs/web/control-ui.md` ・ https://docs.openclaw.ai/ja-JP/web/control-ui

## 一言まとめ

Gateway が配信する **Vite + Lit のシングルページアプリ**で、同じポートの WS と直接話す**ブラウザー管理ダッシュボード**。チャット・音声会話・チャネル・セッション・Cron・Skills・ノード・exec 承認・設定編集・ログ・更新まで、Gateway 運用の大半をブラウザーから行える。

## 位置づけ

[[components/control-ui]] の中核ソース。[[components/webchat]]（ネイティブチャット UI）や [[components/cli]]（端末）と並ぶ「Gateway クライアント」の一つで、初回接続は [[concepts/pairing]]、設定編集は [[concepts/configuration]]、外から開く手段は [[concepts/remote-access]]。

## 仕組み・ふるまい

- **認証**：WS ハンドシェイクで `connect.params.auth.token`/`password`、または Tailscale Serve / trusted-proxy の ID ヘッダー。ランタイム設定は `/__openclaw/control-ui-config.json`（同じ Gateway 認証で保護）。
- **デバイスペアリング**：新ブラウザー/デバイスは一度限りの承認（`"pairing required" (1008)`、`openclaw devices list|approve`）。direct loopback（`127.0.0.1`）は自動承認、読み取り→書き込み/admin の昇格は再承認。各ブラウザープロファイルは固有デバイス ID。
- **できること**：チャット（`chat.history`/`send`/`abort`/`inject`、ブラウザー Talk＝[[concepts/voice]]）、チャネル/インスタンス/セッション/Dreaming、Cron/Skills/ノード/exec 承認、設定（`config.get/set/apply/patch`・スキーマフォーム・SecretRef 事前確認）、デバッグ/ログ/更新。
- **PWA + Web Push**：`manifest.webmanifest`＋service worker でインストール可、VAPID 鍵で push 通知（タブ未起動でも）。

## 設定・使い方の要点

- `gateway.controlUi.*`：`enabled`/`basePath`/`allowedOrigins`/`embedSandbox`（strict/scripts/trusted）/`chatMessageMaxWidth`/`allowExternalEmbedUrls`。ビルドは `pnpm ui:build`、dev は `pnpm ui:dev` ＋ `?gatewayUrl=...#token=...`。
- 個人 ID・テーマ（tweakcn）・言語はブラウザーローカル（Gateway 設定に書かない）。アバター/assistant-media は認証付きルート＋短命 `mediaTicket`、CSP で `img-src` を same-origin/`data:`/`blob:` に限定。

## 注意点・落とし穴

- ⚠️ **プレーン HTTP は非セキュアコンテキスト**で WebCrypto がブロックされ、デバイス ID なし接続は既定でブロック。回避は HTTPS（Tailscale Serve）かローカル（`127.0.0.1`）。互換トグル `allowInsecureAuth`（localhost 限定）、緊急の `dangerouslyDisableDeviceAuth`（重大な低下）。
- 非 loopback デプロイは `allowedOrigins`（完全オリジン）必須。`allowedOrigins: ["*"]` は使わない。

## 用語と略称

- **SPA** = Single Page Application（単一ページの Web アプリ）
- **PWA** = Progressive Web App（インストール可能な Web アプリ）
- **VAPID** = Web Push の署名鍵方式
- **CSP** = Content Security Policy（読み込み元を制限する仕組み）
- **mediaTicket** = メディア URL 用の短命・限定スコープのトークン

## 関連ページ

- [[components/control-ui]] — 対応する構成要素ページ
- [[sources/web/web]] / [[sources/web/webchat]] / [[sources/web/tui]]
- [[concepts/pairing]] / [[concepts/configuration]] / [[concepts/voice]] / [[concepts/remote-access]]
