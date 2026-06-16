---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/browser-linux-troubleshooting
source_path: raw/docs/tools/browser-linux-troubleshooting.md
doc_section: tools
title: "ブラウザーのトラブルシューティング（Linux）"
ingested: 2026-06-16
tags: [browser, linux, troubleshooting, chrome, cdp, snap, headless]
related:
  - "[[components/browser]]"
  - "[[concepts/configuration]]"
---

# ブラウザーのトラブルシューティング（Linux）（解説）

> 原典: `raw/docs/tools/browser-linux-troubleshooting.md` ・ https://docs.openclaw.ai/ja-JP/tools/browser-linux-troubleshooting

## 一言まとめ

Linux で `browser` 制御サーバーが Chrome/Brave/Edge/Chromium の **CDP（Chrome DevTools Protocol）起動に失敗する**典型問題（`Failed to start Chrome CDP on port 18800`）の原因と直し方。中心的な犯人は **Ubuntu 等の snap 版 Chromium**。

## 位置づけ

[[components/browser]] を Linux ホストで動かすときの実務ガイド。設定キーは [[concepts/configuration]] の `browser.*` に属する。

## 仕組み・ふるまい（なぜ失敗するか）

Ubuntu 等の既定 Chromium は **snap パッケージ**で、Snap の **AppArmor confinement（プロセスを隔離する Linux のセキュリティ機構）**が OpenClaw によるブラウザープロセスの生成・監視に干渉する。`apt install chromium` は snap にリダイレクトするスタブを入れるだけで実ブラウザーではない。その他の典型失敗：

- `The profile appears to be in use by another Chromium process`：管理プロファイルに古い `Singleton*` ロックが残存。OpenClaw は停止済み/別ホストを指すロックを削除して 1 回だけ再試行する。
- `Missing X server or $DISPLAY`：デスクトップセッションの無いホストで表示ありブラウザーを要求した。既定では `DISPLAY`/`WAYLAND_DISPLAY` 両方が未設定なら**ヘッドレスにフォールバック**する。headed override を消すか、`OPENCLAW_BROWSER_HEADLESS=1`、`Xvfb` 起動、`openclaw browser start --headless` のいずれかで対処。

## 設定・使い方の要点（2 つの解決策）

- **解決策 1（推奨）：公式 Google Chrome を入れる**。snap でサンドボックス化されていない `.deb` を入れ、設定で実体を指す：

  ```json
  { "browser": { "enabled": true, "executablePath": "/usr/bin/google-chrome-stable", "headless": true, "noSandbox": true } }
  ```
- **解決策 2：snap Chromium を attach-only で使う**。手動起動した Chromium（`--remote-debugging-port=18800 --user-data-dir=...`）に OpenClaw をアタッチさせる（`browser.attachOnly: true`）。任意で systemd ユーザーサービスにして自動起動。

確認は loopback API を叩く：`curl -s http://127.0.0.1:18791/ | jq '{running,pid,chosenBrowser}'` → `POST /start` → `GET /tabs`。

**主な設定キー（[[concepts/configuration]]）**：`browser.enabled`・`executablePath`・`headless`・`noSandbox`（`--no-sandbox` 付与、一部 Linux で必要）・`attachOnly`・`cdpPort`（既定 18800）・`localLaunchTimeoutMs`（既定 15000）・`localCdpReadyTimeoutMs`（既定 8000）。`OPENCLAW_BROWSER_HEADLESS` はプロセス単位のヘッドレス override。

## 注意点・落とし穴

- Raspberry Pi/低速ストレージでは Chrome が CDP を公開するまで遅いことがある→ `localLaunchTimeoutMs`/`localCdpReadyTimeoutMs` を増やす（120000ms 以下の正整数、無効値は拒否）。
- `No Chrome tabs found for profile="user"`：existing-session/Chrome MCP プロファイルで開いているタブが無い。管理プロファイル `openclaw` を使うか、ローカル Chrome に最低 1 タブを開いて `--browser-profile user` で再試行。`user` は**ホスト専用**で機能制限あり（ref 駆動のみ、`responsebody`/PDF/ダウンロード傍受/バッチ不可）。
- リモート CDP プロファイルは `http(s)://`（`/json/version` 検出用）/`ws(s)://`（直接ソケット）を受ける。

## 用語と略称

- **CDP** = Chrome DevTools Protocol（ブラウザーを遠隔操作する規格）
- **snap** = Ubuntu のサンドボックス型パッケージ形式（AppArmor で隔離され OpenClaw の起動制御を妨げる）
- **attach-only** = OpenClaw がブラウザーを起動せず、既に動いているものへ接続だけする
- **headless** = GUI 無しのブラウザー起動 / **Xvfb** = 仮想ディスプレイサーバー
- **MCP** = Model Context Protocol（Chrome DevTools MCP で existing-session に接続）

## 関連ページ

- [[components/browser]] / [[concepts/configuration]]
- [[sources/tools/browser]] / [[sources/tools/browser-control]] / [[sources/tools/browser-login]]
- [[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]]
