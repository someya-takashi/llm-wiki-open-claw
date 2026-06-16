---
type: component
aliases: [Control UI, 制御 UI, ダッシュボード, Dashboard]
tags: [control-ui, dashboard, browser, spa, web, pwa]
concepts:
  - "[[concepts/architecture]]"
  - "[[concepts/pairing]]"
  - "[[concepts/configuration]]"
  - "[[concepts/remote-access]]"
related:
  - "[[components/gateway]]"
  - "[[components/webchat]]"
  - "[[components/cli]]"
sources:
  - "[[sources/web/control-ui]]"
  - "[[sources/web/web]]"
updated: 2026-06-14
---

# Control UI（制御 UI）

**Control UI** は、[[components/gateway]] が **WS と同じポート**から配信する **Vite + Lit のシングルページアプリ**（既定 `http://<host>:18789/`）。ブラウザーから Gateway を運用する主役のダッシュボードで、[[components/webchat]]（ネイティブチャット）・[[components/cli]]（端末）と並ぶ「Gateway クライアント」の一つ。すべて [[concepts/architecture]] の WS クライアントとして、同じ認証・[[concepts/pairing]] を通る。

## 何ができるか

ブラウザー一つで Gateway 運用の大半を覆う：

- **チャット/音声**：`chat.*`（履歴/送信/中止/注入）、ブラウザー Talk（[[concepts/voice]]）、ツール出力カードのストリーミング。
- **運用**：チャネル（QR ログイン/設定）、インスタンス（[[concepts/presence]]）、セッション（モデル/thinking 上書き）、Dreaming、Cron、Skills、ノード一覧、**exec 承認**（[[concepts/sandboxing]]）。
- **設定**：`~/.openclaw/openclaw.json` の表示/編集（`config.get/set/apply/patch`）、スキーマ駆動フォーム、SecretRef の事前解決確認、ベースハッシュガード（[[concepts/configuration]]・[[concepts/secrets]]）。
- **観測/保守**：ステータス/ヘルス/モデルのスナップショット、ログのライブテール（`logs.tail`）、パッケージ/git 更新（`update.run`）。

## 認証とアクセス

- WS ハンドシェイクで `connect.params.auth.token`/`password`、または Tailscale Serve / trusted-proxy の ID ヘッダー（[[concepts/authentication]]）。新ブラウザー/デバイスは一度限りの [[concepts/pairing]]（`"pairing required" (1008)` → `openclaw devices approve`）。direct loopback は自動承認、読み取り→書き込み/admin 昇格は再承認。
- ⚠️ **プレーン HTTP は非セキュアコンテキスト**で WebCrypto がブロックされ、デバイス ID なし接続は既定ブロック。回避は HTTPS（Tailscale Serve）かローカル（`127.0.0.1`）。非 loopback は `gateway.controlUi.allowedOrigins` 必須。外から開く運用は [[concepts/remote-access]]。
- **PWA + Web Push**：インストール可能で、タブ未起動でも push 通知（VAPID）。

## 位置づけ

詳細・設定キー・dev サーバ手順は [[sources/web/control-ui]]、Web サーフェス全体とバインドモードは [[sources/web/web]]。Gateway が提供する Canvas/A2UI ホスト（`/__openclaw__/canvas/`・`/__openclaw__/a2ui/`）も同じ HTTP サーフェス上にある。

## 関連

- [[concepts/architecture]] / [[concepts/pairing]] / [[concepts/configuration]] / [[concepts/remote-access]] / [[concepts/authentication]]
- [[components/gateway]] / [[components/webchat]] / [[components/cli]]
- [[sources/web/control-ui]] / [[sources/web/web]]
