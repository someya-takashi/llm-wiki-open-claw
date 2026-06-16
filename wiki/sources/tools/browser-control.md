---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/browser-control
source_path: raw/docs/tools/browser-control.md
doc_section: tools
title: "ブラウザー制御 API"
ingested: 2026-06-16
tags: [browser, cli, http-api, playwright, cdp, snapshot, ref, automation]
related:
  - "[[components/browser]]"
  - "[[concepts/session-tool]]"
  - "[[concepts/security]]"
---

# ブラウザー制御 API（解説）

> 原典: `raw/docs/tools/browser-control.md` ・ https://docs.openclaw.ai/ja-JP/tools/browser-control

## 一言まとめ

ブラウザー制御の**リファレンス**ページ――ローカル制御 **HTTP API**、`openclaw browser` **CLI**、そして「スナップショット → ref で要素を掴む → クリック/入力 → 待機 → デバッグ」というスクリプト作成パターンを網羅する。セットアップや設定は [[sources/tools/browser]] 側。

## 位置づけ

[[components/browser]] の「どう叩くか」を定義する層。エージェントの [[concepts/session-tool]] `browser` ツールも、CLI も、HTTP API も、すべて同じ Gateway 内 loopback（ローカル限定）制御サービスに行き着く。3 つの入口は同じ機能の別フェイス。

## 仕組み・ふるまい

小さな loopback 制御サーバーが HTTP を受け、**CDP（Chrome DevTools Protocol, ブラウザーを遠隔操作する規格）**で Chromium 系ブラウザーへ接続する。クリック/入力/スナップショット/PDF などの高度操作は **CDP 上の Playwright（ブラウザー自動化ライブラリ）**を経由する。Playwright が無い環境では、Playwright 不要の操作（ARIA スナップショット、CDP WebSocket があればロールスナップショットやページスクリーンショット）のみ使える。

### スナップショットと ref（中核概念）

エージェントは **CSS セレクターを使わず**、スナップショットが返す **ref（要素の参照 ID）**で対象を指す。3 形式ある：

- **AI スナップショット（数値 ref `12`）**：既定（`--format ai`）。内部で Playwright の `aria-ref` に解決。`click 12`・`type 23 "hello"`。
- **ロールスナップショット（ロール ref `e12`）**：`--interactive`/`--compact`/`--depth`/`--selector`/`--frame`。`getByRole(...)` で解決（重複は `nth()`）。`--labels` で viewport スクショに `e12` ラベルを重ねられる。
- **ARIA スナップショット（`ax12`）**：`--format aria`。アクセシビリティツリー。Playwright があればライブページにバインドされアクション可能、無ければ**検査専用**。

⚠️ **ref はナビゲーションをまたいで安定しない**。失敗したら `snapshot` を撮り直して新しい ref を使う。古い `axN` は即座に失敗する（フォールスルーしない）。タブ ID/ラベルは（置換先を証明できれば）安定するので、スクリプトは `suggestedTargetId` を優先。

## 設定・使い方の要点

- **HTTP 制御 API**：`GET /`（ステータス）・`POST /start|/stop`・`/tabs*`・`GET /snapshot`・`POST /screenshot`・`POST /navigate`・`POST /act`・Cookie/ストレージ/`set/*`（offline・headers・geolocation・device 等）・デバッグ（`/console`・`/errors`・`/requests`・`/trace/*`）。全エンドポイントは `?profile=<name>` を受ける。`POST /start?headless=true` は管理プロファイルの一度限りヘッドレス起動（attach-only/リモート CDP/既存セッションは起動しないので拒否）。
  - 共有シークレット Gateway 認証時は `Authorization: Bearer <token>` か `x-openclaw-password`/HTTP Basic が要る。⚠️ trusted-proxy/Tailscale のアイデンティティは継承しない＝**ループバック専用**に保つ。
- **`/act` エラー契約**：`{ "error": "...", "code": "ACT_*" }`（`ACT_KIND_REQUIRED`/`ACT_INVALID_REQUEST`/`ACT_SELECTOR_UNSUPPORTED`/`ACT_EVALUATE_DISABLED`(403)/`ACT_TARGET_ID_MISMATCH`(403)/`ACT_EXISTING_SESSION_UNSUPPORTED`(501)）。
- **CLI（`openclaw browser ...`）**：`status`/`start [--headless]`/`stop`/`tabs`/`open`/`focus`/`close`、検査（`screenshot [--ref|--full-page|--labels]`・`snapshot [--format|--interactive|--efficient|--selector|--frame]`・`console`・`errors`・`requests`・`pdf`・`responsebody`）、アクション（`navigate`・`click [--double]`・`click-coords`・`type --submit`・`press`・`hover`・`drag`・`select`・`download`・`upload`・`fill`・`dialog`・`wait`・`evaluate`・`highlight`・`trace start|stop`）、状態（`cookies`・`storage`・`set offline|headers|credentials|geo|media|timezone|locale|device|viewport`）。全コマンドが `--browser-profile <name>` と `--json` を受ける。
- **待機の強化**：時間/テキストに加え、URL（glob）・読み込み状態（`networkidle`）・JS 述語（`--fn`）・セレクター表示を待て、組み合わせ可。
- **準備呼び出し**：`upload`/`dialog` は chooser/dialog を起こすクリックの**前**に実行する。
- **パス制限**：ダウンロード/トレース/アップロードは `/tmp/openclaw{,/downloads,/uploads}` に限定。

### Playwright 要件と Docker

navigate/act/AI スナップショット/要素スクショ/PDF は Playwright 必須（無いと明確な 501）。`Playwright is not available in this gateway build` は再インストール/更新で解消。Docker では `npx playwright` を避け、`OPENCLAW_INSTALL_BROWSER=1 ./scripts/docker/setup.sh` か `playwright-core/cli.js install chromium` で Chromium を入れ、`PLAYWRIGHT_BROWSERS_PATH` を永続ボリュームに置く。

## 注意点・落とし穴

- ⚠️ **任意 JS 実行**：`evaluate` と `wait --fn` はページ内で任意 JavaScript を走らせる。プロンプトインジェクション経路になりうるので不要なら `browser.evaluateEnabled: false`（[[concepts/security]]・[[concepts/threat-model]]）。
- ⚠️ **SSRF ポリシー**：既定でプライベート/内部宛先をブロック。`browser.ssrfPolicy.hostnameAllowlist` 等で明示許可（[[sources/security/network-proxy]]）。
- ⚠️ **ログイン済みプロファイルは機密**。リモート CDP エンドポイントは強力なのでトンネルで保護。
- CSS セレクターはアクションでは意図的に非サポート。viewport 位置しか頼れない時は `click-coords`。

## 用語と略称

- **CDP** = Chrome DevTools Protocol（ブラウザーを遠隔操作する規格）
- **Playwright** = ブラウザー自動化ライブラリ（CDP 上で動く主要アクションエンジン）
- **ref** = スナップショットが返す要素参照 ID（数値 `12` / ロール `e12` / ARIA `ax12`）
- **loopback** = `127.0.0.1` だけで待ち受けるローカル限定の待ち受け
- **SSRF** = Server-Side Request Forgery（内部アドレスへの不正ナビゲーション）
- **headless** = GUI 無しのブラウザー起動

## 関連ページ

- [[components/browser]] / [[concepts/session-tool]] / [[concepts/security]] / [[concepts/threat-model]]
- [[sources/tools/browser]] / [[sources/tools/browser-login]] / [[sources/security/network-proxy]]
- [[sources/tools/browser-linux-troubleshooting]] / [[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]]
