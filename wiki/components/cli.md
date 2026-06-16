---
type: component
aliases: [CLI, Command Line Interface, openclaw, TUI, ターミナル UI, openclaw chat]
tags: [cli, tui, terminal, command-line, local-mode, client]
concepts:
  - "[[concepts/architecture]]"
  - "[[concepts/configuration]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/session]]"
related:
  - "[[components/gateway]]"
  - "[[components/control-ui]]"
  - "[[components/webchat]]"
sources:
  - "[[sources/web/tui]]"
updated: 2026-06-14
---

# CLI（コマンドラインインターフェース）

**CLI** は `openclaw` コマンド群＝OpenClaw を端末から操作する主要インターフェース。本 wiki の至るところで使われる `openclaw gateway`（[[components/gateway]] 起動）・`openclaw config`（[[concepts/configuration]]）・`openclaw devices`（[[concepts/pairing]]）・`openclaw nodes`（[[components/node]]）・`openclaw doctor`（[[concepts/diagnostics]]）などはすべてこの CLI のサブコマンド。そのうち**対話的なチャット UI が TUI**（`openclaw tui`）で、[[components/control-ui]]（ブラウザー）・[[components/webchat]]（ネイティブ）と並ぶ Gateway クライアントになる。

## TUI（ターミナル UI）

`openclaw tui` は端末上のフルスクリーンなチャットシェル。2 形態を持つ：

- **Gateway モード**（`openclaw tui`）：起動中の Gateway に WS 接続（`mode: "tui"` で登録、`--url`/`--token`/`--password`）。[[concepts/architecture]] の WS クライアント。
- **ローカルモード**（`openclaw chat` ＝ `openclaw tui --local`）：**Gateway を介さず**[[concepts/agent-runtimes]] の埋め込みランタイムを直接使う。Gateway 専用機能は不可だが、設定が壊れていても動くので**設定修復**に有用。

セッションキーは `agent:<agentId>:<sessionKey>`（[[concepts/session]]）で、スラッシュコマンド（`/session`・`/model`・`/think`・`/deliver`・`/elevated` など）と行頭 `!` のローカルシェル実行をサポート。配信は既定オフ（`/deliver on` で実チャネルへ）。詳細・ショートカット・オプションは [[sources/web/tui]]。

## なぜ重要か

CLI は「ヘッドレス運用の唯一の操作面」であり、GUI が無いサーバー/コンテナでも Gateway の起動・設定・ペアリング・診断をすべて行える。ローカル TUI は Gateway に依存せず埋め込みランタイムで走るため、**Gateway が壊れていても自己診断・修復できる**安全弁になる（`!openclaw config validate` / `doctor --fix`）。

## 位置づけ

本ページは対話 UI（TUI）を中心に、CLI コマンド群の入口を兼ねる。各サブコマンドの詳細は対応する concept/source（例 `openclaw secrets`→[[concepts/secrets]]、`openclaw security audit`→[[sources/gateway/security/audit-checks]]）に分散。`cli/` セクションの専用ドキュメントは今後の ingest で拡充する。

## 関連

- [[concepts/architecture]] / [[concepts/configuration]] / [[concepts/agent-runtimes]] / [[concepts/session]] / [[concepts/slash-commands]]
- [[components/gateway]] / [[components/control-ui]] / [[components/webchat]] / [[components/node]]
- [[sources/web/tui]] / [[sources/tools/slash-commands]] / [[sources/tools/agent-send]]（`openclaw agent`）
