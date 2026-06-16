---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/web/tui
source_path: raw/docs/web/tui.md
doc_section: web
title: "TUI（ターミナル UI）"
ingested: 2026-06-14
tags: [tui, terminal, cli, local-mode, gateway-mode, slash-commands]
related:
  - "[[components/cli]]"
  - "[[concepts/session]]"
  - "[[concepts/agent-runtimes]]"
---

# TUI（ターミナル UI）（解説）

> 原典: `raw/docs/web/tui.md` ・ https://docs.openclaw.ai/ja-JP/web/tui

## 一言まとめ

端末上で動く対話的なチャットシェル `openclaw tui`。**Gateway モード**（起動中の Gateway に WS 接続）と**ローカルモード**（Gateway なしで埋め込みエージェントランタイムを直接使う、`openclaw chat`）の 2 形態を持つ。

## 位置づけ

[[components/cli]] の対話 UI 面。[[components/control-ui]]（ブラウザー）・[[components/webchat]]（ネイティブ）と並ぶ Gateway クライアントで、ローカルモードは Gateway を介さず [[concepts/agent-runtimes]] の埋め込みランタイムで走る（設定修復などに有用）。

## 仕組み・ふるまい

- **2 モード**：`openclaw tui`（Gateway、`--url`/`--token`/`--password`）／`openclaw chat`＝`openclaw tui --local`（ローカル、`--url` 等と併用不可、Gateway 専用機能は不可）。`mode: "tui"` として Gateway に登録。
- **エージェント+セッション**：セッションキーは `agent:<agentId>:<sessionKey>`。`/session main` は現在エージェントに展開、`/session agent:other:main` で明示切替。スコープ per-sender（既定）/global。フッターに接続/エージェント/セッション/モデルを常時表示。
- **配信は既定オフ**：メッセージは Gateway に送るがプロバイダー配信はしない（`/deliver on` で有効）。
- **ローカルシェル**：行頭 `!` で TUI ホスト上のローカルシェル実行（セッションごとに 1 回許可確認、`OPENCLAW_SHELL=tui-local`）。

## 設定・使い方の要点

- スラッシュコマンド：`/help`・`/status`・`/agent`・`/session`・`/model`、セッション制御（`/think`・`/fast`・`/verbose`・`/trace`・`/reasoning`・`/elevated`・`/activation`・`/deliver`）、`/new`・`/reset`・`/abort`。ローカルのみ `/auth`。他は Gateway に転送。
- ショートカット：Enter 送信・Esc 中止・Ctrl+L/G/P（モデル/エージェント/セッションピッカー）・Ctrl+O（ツール出力展開）・Ctrl+T（思考表示）。`--history-limit`（既定 200）・`OPENCLAW_THEME=light|dark`。

## 注意点・落とし穴

- ⚠️ `--url` 指定時は config/env 認証にフォールバックしない（`--token`/`--password` 必須）。ローカルモードでは `--url`/`--token`/`--password` を渡さない。
- **設定修復ループ**：`openclaw chat` で埋め込みエージェントに現設定をドキュメントと突き合わせさせ、`!openclaw config validate`/`doctor --fix` で直す（無効設定ガードは回避しない）。

## 用語と略称

- **TUI** = Terminal User Interface（端末上の UI）
- **Gateway モード / ローカルモード** = 起動中 Gateway に接続 / Gateway なしで埋め込みランタイム実行
- **deliver** = アシスタント返信を実際のチャネルに配信するか
- **埋め込みエージェントランタイム** = Gateway を介さずローカルで走るエージェント実行（[[concepts/agent-runtimes]]）

## 関連ページ

- [[components/cli]] — 対応する構成要素ページ
- [[sources/web/control-ui]] / [[sources/web/webchat]]
- [[concepts/session]] / [[concepts/agent-runtimes]] / [[concepts/configuration]]
