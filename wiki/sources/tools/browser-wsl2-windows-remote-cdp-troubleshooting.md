---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/browser-wsl2-windows-remote-cdp-troubleshooting
source_path: raw/docs/tools/browser-wsl2-windows-remote-cdp-troubleshooting.md
doc_section: tools
title: "WSL2 + Windows + リモート Chrome CDP のトラブルシューティング"
ingested: 2026-06-16
tags: [browser, wsl2, windows, remote-cdp, troubleshooting, control-ui, networking]
related:
  - "[[components/browser]]"
  - "[[concepts/remote-access]]"
  - "[[components/control-ui]]"
---

# WSL2 + Windows + リモート Chrome CDP のトラブルシューティング（解説）

> 原典: `raw/docs/tools/browser-wsl2-windows-remote-cdp-troubleshooting.md` ・ https://docs.openclaw.ai/ja-JP/tools/browser-wsl2-windows-remote-cdp-troubleshooting

## 一言まとめ

**Gateway が WSL2、Chrome が Windows** という分割ホスト構成で、ブラウザー制御が WSL2/Windows 境界を越える時の段階的トラブルシュート。難しさの本質は「ブラウザー転送・Control UI のオリジン認証・トークン/ペアリングが**独立に失敗**しながら、ユーザーからは似たエラーに見える」こと。

## 位置づけ

[[components/browser]] のリモート CDP（[[concepts/remote-access]]）構成の中でも、WSL2（Windows 上の Linux 環境）特有の分割ホスト問題。[[components/control-ui]] の insecure-origin 認証問題が混線しやすい点が肝。

## 仕組み・ふるまい（まずモードを選ぶ）

2 つの有効パターンがある：

- **オプション 1（推奨）：WSL2 → Windows の raw remote CDP**。WSL2 から Windows Chrome の **CDP（Chrome DevTools Protocol）**エンドポイントを指すリモートプロファイルを使う。Gateway が WSL2・Chrome が Windows・境界越えが要る場合。
- **オプション 2：ホストローカル Chrome MCP（`existing-session`/`user`）**。Gateway と Chrome が**同一ホスト**の時だけ。`responsebody`/PDF/ダウンロード傍受/バッチのような managed/raw-CDP 専用ルートは使えない。

⚠️ Chrome MCP は**ホストローカル**であって WSL2→Windows ブリッジではない。WSL2 Gateway＋Windows Chrome では raw remote CDP を使う。

参照アーキテクチャ：WSL2 が `127.0.0.1:18789` で Gateway を実行 → Windows のブラウザーで `http://127.0.0.1:18789/` の Control UI を開く → Windows Chrome がポート `9222` で CDP を公開 → WSL2 からその CDP に到達でき、OpenClaw のブラウザープロファイルが WSL2 から到達可能なアドレスを指す。

## 設定・使い方の要点（階層ごとに上から検証）

飛ばさず上から：

1. **Chrome が Windows で CDP を出しているか**：Windows で `chrome.exe --remote-debugging-port=9222` を起動し、`curl http://127.0.0.1:9222/json/version`・`/json/list` を Windows 側で確認。ここで失敗ならまだ OpenClaw の問題ではない。
2. **WSL2 からその Windows エンドポイントに到達できるか**：WSL2 から `curl http://WINDOWS_HOST_OR_IP:9222/json/version` をテスト。失敗ならポート公開/アドレス/ファイアウォール/ポート転送の問題。**OpenClaw 設定を触る前に直す**。
3. **正しいブラウザープロファイルを設定**：WSL2 から到達可能なアドレスを `cdpUrl` に。`attachOnly: true` を維持：

   ```json5
   { browser: { enabled: true, defaultProfile: "remote",
     profiles: { remote: { cdpUrl: "http://WINDOWS_HOST_OR_IP:9222", attachOnly: true, color: "#00AA00" } } } }
   ```
   `cdpUrl` は `http(s)://`（`/json/version` 検出用）/`ws(s)://`（直接ソケット）。期待する前に同じ URL を `curl` でテスト。
4. **Control UI レイヤーを別に確認**：Windows から `http://127.0.0.1:18789/` を開き、ページオリジンが `gateway.controlUi.allowedOrigins` と一致・トークン/ペアリングが正しいかを確認（[[components/control-ui]]）。
5. **E2E のブラウザー制御**：WSL2 から `openclaw browser open https://example.com --browser-profile remote` → `tabs`。タブが Windows Chrome で開けば成功。

**Control UI の重要ルール**：Windows から UI を開く時は LAN IP でなく **`http://127.0.0.1:18789/`** を使う。LAN/tailnet 上のプレーン HTTP は、CDP とは無関係な insecure-origin / device-auth 動作を誘発する。

## 注意点・落とし穴（誤解を招くエラー）

各メッセージを「どの階層の手がかりか」で読む：

- `control-ui-insecure-auth`：UI オリジン/セキュアコンテキストの問題（CDP 転送ではない）。
- `token_missing` / `pairing required`：認証/デバイス承認の問題。
- `Remote CDP for profile "remote" is not reachable`：WSL2 から `cdpUrl` に到達できない。
- `Browser attachOnly ... CDP websocket ... not reachable`：HTTP は応答したが DevTools WebSocket を開けない。
- リモートセッション後の古い viewport/dark-mode/locale/offline override → `openclaw browser stop --browser-profile remote`（Gateway や外部ブラウザーを再起動せず制御セッションを閉じてエミュレーション状態を解放）。
- `gateway timeout after 1500ms`：多くは CDP 到達性 or 低速/到達不能なリモートエンドポイント。
- `No Chrome tabs found for profile="user"`：ホストローカルのタブが無い場所でローカル Chrome MCP プロファイルを選んでいる。

**高速トリアージ**：① Windows で `:9222/json/version` が動くか → ② WSL2 から同じが動くか → ③ `cdpUrl` がその WSL2 到達アドレスか → ④ Control UI を `127.0.0.1:18789` で開いているか → ⑤ 境界越えに `existing-session` を誤用していないか。

## 用語と略称

- **WSL2** = Windows Subsystem for Linux 2（Windows 上で動く Linux 環境）
- **CDP** = Chrome DevTools Protocol（ブラウザーを遠隔操作する規格） / **MCP** = Model Context Protocol
- **raw remote CDP** = 別ホストの Chrome の CDP エンドポイントへ直接アタッチするプロファイル
- **attach-only** = OpenClaw がブラウザーを起動せず接続のみ
- **insecure origin** = HTTPS でない/localhost でないオリジン（セキュアコンテキスト扱いされず認証挙動が変わる）

## 関連ページ

- [[components/browser]] / [[concepts/remote-access]] / [[components/control-ui]]
- [[sources/tools/browser]] / [[sources/tools/browser-control]] / [[sources/tools/browser-login]] / [[sources/tools/browser-linux-troubleshooting]]
