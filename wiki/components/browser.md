---
type: component
aliases: [Browser, ブラウザー制御, Browser Control, browser tool]
tags: [browser, chrome, cdp, playwright, automation, profile, ssrf]
concepts:
  - "[[concepts/session-tool]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/security]]"
related:
  - "[[components/gateway]]"
  - "[[components/node]]"
  - "[[components/plugin-system]]"
sources:
  - "[[sources/tools/browser]]"
  - "[[sources/tools/browser-control]]"
  - "[[sources/tools/browser-login]]"
  - "[[sources/tools/browser-linux-troubleshooting]]"
  - "[[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]]"
updated: 2026-06-16
---

# ブラウザー制御（Browser）

**ブラウザー制御**は、エージェントが**実際の Chromium 系ブラウザー（Chrome/Brave/Edge/Chromium）を操作する**ためのサブシステム。タブを開く・ページを読む（スナップショット）・クリック・入力・ダウンロード・スクリーンショットといった「人間がブラウザーでやること」をエージェントに与える。実体は [[components/gateway]] 内の小さな **loopback（ループバック, `127.0.0.1` だけで待ち受けるローカル限定）制御サービス**と、同梱の `browser` Plugin（[[components/plugin-system]]、既定有効）で、エージェントからは [[concepts/session-tool]] の `browser` ツールとして 1 つの安定インターフェイスに見える。

## なぜ重要か

Web 検索（[[concepts/web-search]]）が「読む」中心なのに対し、ブラウザー制御は**ログインの要るサイトや動的な UI を実際に操作できる**――X/Twitter への投稿、SaaS ダッシュボードの操作、フォーム入力などが可能になる。その分**強力で危険**でもあり、ログイン済みセッションを握り、任意ナビゲーション・任意 JS 実行の面を持つため、[[concepts/security]]・[[concepts/threat-model]] は**ブラウザー制御をオペレーターアクセス相当**として扱う。「1 つの安定インターフェイスの下でローカル/リモートのブラウザーとプロファイルが自由に入れ替わる」設計が、運用の柔軟さ（手元の Chrome／リモート CDP／クラウド）を与えつつ、エージェント側のツール契約を一定に保つ。

## プロファイル（どのブラウザーを操作するか）

ブラウザー制御の挙動は**プロファイル**で決まる。誤って個人用ブラウザーを操作しないための隔離が核心。

- **`openclaw`（既定）**：エージェント専用の隔離プロファイル（オレンジ色の UI）。個人用ブラウザーに触れない。OpenClaw が起動・管理する。
- **`user`（existing-session / Chrome DevTools MCP）**：ユーザーが実際にサインイン済みのローカル Chrome に、Chrome DevTools MCP（Model Context Protocol, 外部ツールをエージェントに接続する規格）経由でアタッチする。**ホスト専用**で、機能制限あり（ref 駆動アクションのみ、`responsebody`/PDF/ダウンロード傍受/バッチ不可）。
- **リモート CDP プロファイル**：別ホストの Chrome の **CDP（Chrome DevTools Protocol, ブラウザーを遠隔操作する規格）**エンドポイントに `cdpUrl` で接続（`attachOnly: true`）。Node ブラウザープロキシ（設定不要の既定）／Browserless（ホスト型リモート CDP）／直接 WebSocket CDP（Browserbase 等）。WSL2 Gateway＋Windows Chrome のような分割ホストはこれを使う。

## サーフェス（3 つの入口）

同じ制御サービスを 3 通りで叩ける：

- **エージェントの `browser` ツール**：[[concepts/session-tool]] 経由。通常の使い方。
- **`openclaw browser` CLI**：`status`/`start`/`open`/`snapshot`/`click`/`type`/`navigate` など。`--browser-profile <name>` でプロファイル指定、`--json` で機械可読出力。詳細 [[sources/tools/browser-control]]。
- **loopback HTTP 制御 API（任意）**：`POST /start`・`/navigate`・`/act`・`/snapshot` など。ローカル統合専用で、共有シークレット Gateway 認証が有効ならこの HTTP ルートにも認証が要る。⚠️ trusted-proxy/Tailscale のアイデンティティヘッダーは継承しないため**ループバック専用に保つ**。

## スナップショットと ref（エージェントがページを「掴む」方法）

エージェントは CSS セレクターではなく、**スナップショットが返す ref（参照 ID）**で要素を操作する：AI スナップショット（数値 `12`）／ロールスナップショット（`e12`）／ARIA スナップショット（`ax12`）。高度なアクション（navigate/act/AI スナップショット/PDF）は **Playwright（ブラウザー自動化ライブラリ）**を必要とし、無い場合は ARIA スナップショット等の限定機能のみ。⚠️ **ref はナビゲーションをまたいで安定しない**ので、失敗したら撮り直す。

## セキュリティ上の位置づけ

- ⚠️ **SSRF（Server-Side Request Forgery, サーバーに内部アドレスへ不正リクエストさせる攻撃）防御**：内部/プライベート宛先へのナビゲーションは既定でブロック（`browser.ssrfPolicy`、[[sources/security/network-proxy]]）。
- ⚠️ **任意 JS 実行**：`evaluate` と `wait --fn` はページ内で任意 JavaScript を実行する。プロンプトインジェクションに悪用されうるので、不要なら `browser.evaluateEnabled: false`。
- ⚠️ **ログイン済みセッションは機密**：`openclaw` プロファイルにサインイン状態が残るため機密扱い。自動ログインはボット対策を誘発するので**手動ログイン推奨**（[[sources/tools/browser-login]]）。
- Gateway/Node ホストは非公開（ループバックまたは tailnet のみ）に保ち、リモート CDP エンドポイントはトンネルで保護する。サンドボックス（[[concepts/sandboxing]]）有効時はブラウザーも既定でサンドボックス側になり、ホスト制御には `sandbox.browser.allowHostControl: true` が要る。

## トラブルシューティングの勘所

- **Linux**：Ubuntu 等の既定 Chromium は **snap パッケージ**で AppArmor 制限が干渉する。公式 Google Chrome `.deb` を入れるか、attach-only で手動起動した Chromium にアタッチする（[[sources/tools/browser-linux-troubleshooting]]）。
- **WSL2＋Windows**：Gateway（WSL2）と Chrome（Windows）が境界を跨ぐ分割構成は、ブラウザー転送・Control UI のオリジン認証・トークン/ペアリングが**独立に失敗**して切り分けが難しい。レイヤー（Chrome→WSL2 到達性→プロファイル→Control UI→E2E）を上から順に検証する（[[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]]）。

## 代表ソース

- [[sources/tools/browser]] — 概要・プロファイル・設定・セキュリティ
- [[sources/tools/browser-control]] — 制御 HTTP API・`openclaw browser` CLI・スナップショット/ref・デバッグフロー
- [[sources/tools/browser-login]] — 手動ログインと X/Twitter フロー
- [[sources/tools/browser-linux-troubleshooting]] — Linux の CDP 起動問題
- [[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]] — WSL2＋Windows のリモート CDP

## 関連

- [[concepts/session-tool]]（ツールとしての面） / [[concepts/sandboxing]] / [[concepts/security]] / [[concepts/threat-model]]
- [[concepts/remote-access]]（リモート CDP・トンネル） / [[concepts/web-search]]（読む側との対比）
- [[components/gateway]]（制御サービスのホスト） / [[components/node]]（Node ブラウザープロキシ） / [[components/plugin-system]]（`browser` Plugin）
