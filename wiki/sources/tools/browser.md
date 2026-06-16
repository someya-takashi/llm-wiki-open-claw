---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/browser
source_path: raw/docs/tools/browser.md
doc_section: tools
title: "ブラウザー（OpenClaw 管理）"
ingested: 2026-06-14
tags: [browser, chrome, cdp, automation, profile, browserless, ssrf]
related:
  - "[[concepts/session-tool]]"
  - "[[components/plugin-system]]"
  - "[[concepts/sandboxing]]"
---

# ブラウザー（OpenClaw 管理）（解説）

> 原典: `raw/docs/tools/browser.md` ・ https://docs.openclaw.ai/ja-JP/tools/browser

## 一言まとめ

エージェントが制御する**専用の Chrome/Brave/Edge/Chromium プロファイル**を、Gateway 内のローカル制御サービス（loopback のみ）経由で動かす `browser` ツール。個人用ブラウザーから分離された「エージェント専用ブラウザー」で、タブを開く・ページを読む・クリック・入力ができる。

## 位置づけ

[[concepts/session-tool]] のブラウザツールで、実体は同梱の `browser` Plugin（[[components/plugin-system]]、既定有効）。隔離と SSRF 防御は [[concepts/sandboxing]]・[[concepts/security]]、リモート制御は [[concepts/remote-access]] と関連。

## 仕組み・ふるまい

- **2 プロファイル**：`openclaw`（エージェント専用・個人用に触れない隔離プロファイル）と `user`（Chrome DevTools MCP 経由で実際のサインイン済み Chrome に接続）。
- **制御サービス**：Gateway 内の小さな loopback 制御サービス（`browser.request` Gateway メソッド・`openclaw browser` CLI）。
- **リモート CDP**：Node ブラウザープロキシ（設定不要の既定）/ Browserless（ホスト型リモート CDP・Docker）/ 直接 WebSocket CDP（Browserbase 等）。

## 設定・使い方の要点

- `browser.*` 設定。Brave/Edge/Chromium 選択（OS 別の起動パス）。プロファイル（マルチブラウザー）。Chrome DevTools MCP で既存セッションに接続（カスタム Chrome MCP 起動）。
- ブラウザツールが見えない時は Plugin（`browser`）が有効か・制御サービスが起動しているかを確認。

## 注意点・落とし穴

- ⚠️ **隔離の保証**：`openclaw` プロファイルは個人用プロファイルに触れない。⚠️ **ナビゲーション SSRF ブロック**：CDP 起動失敗時のトラブルや、内部アドレスへのナビゲーションは SSRF 防御で弾かれる（[[sources/security/network-proxy]]）。ブラウザ制御はオペレーターアクセス相当に扱う（[[concepts/threat-model]]）。

## 用語と略称

- **CDP** = Chrome DevTools Protocol（ブラウザを遠隔制御する規格）
- **MCP** = Model Context Protocol（Chrome DevTools MCP で既存セッション接続）
- **Browserless / Browserbase** = ホスト型のリモート CDP プロバイダー
- **`openclaw` / `user` プロファイル** = エージェント専用 / 実サインイン済みブラウザー
- **SSRF** = Server-Side Request Forgery（内部アドレスへの不正ナビゲーション）

## 関連ページ

- [[concepts/session-tool]] / [[components/plugin-system]] / [[concepts/sandboxing]]
- [[concepts/security]] / [[concepts/threat-model]] / [[sources/security/network-proxy]] / [[sources/tools/media-overview]]
