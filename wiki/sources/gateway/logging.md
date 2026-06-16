---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/logging
source_path: raw/docs/gateway/logging.md
doc_section: gateway
title: "Gateway のログ記録（内部）"
ingested: 2026-06-14
tags: [logging, ws-log, subsystem, console-capture, redaction]
related:
  - "[[concepts/logging]]"
  - "[[components/gateway]]"
  - "[[sources/logging]]"
---

# Gateway のログ記録（内部）（解説）

> 原典: `raw/docs/gateway/logging.md` ・ https://docs.openclaw.ai/ja-JP/gateway/logging
>
> ℹ️ 利用者向けの概要は [[sources/logging]]（root の `/logging`）。本ページは **Gateway 内部**の WS ログスタイル・サブシステム整形・コンソールキャプチャの詳細。

## 一言まとめ

Gateway のログ実装まわり――ファイルベースロガー（JSONL ローテーション）、**Gateway WebSocket プロトコルログ**の 2 モード、サブシステムプレフィックス付きの TTY 対応コンソール整形、コンソールキャプチャを説明したページ。

## 位置づけ

[[concepts/logging]] の Gateway 実装面。[[sources/logging]] が「どこに・どう読むか」を、本ページが「Gateway がどう出すか」を担う。[[components/gateway]] の運用と密接。

## 仕組み・ふるまい

- **ファイルロガー**：既定 `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（1 日 1 ファイル、`logging.maxFileBytes` 既定 100MB でローテーション＋アーカイブ 5）。1 行 1 JSON。起動時に解決済みデフォルトモデルを記録（例 `agent model: openai-codex/gpt-5.5 (thinking=medium, fast=on)`）。
- **Gateway WebSocket ログ**：通常モード（`--verbose` 無し）は「興味深い」RPC のみ（エラー `ok=false`・遅い呼び出し 既定 `≥50ms`・parse error）、verbose モードは全 req/res。スタイルは `--ws-log auto|compact|full`（`--compact` は compact エイリアス）。
- **コンソール整形**：TTY 対応で**サブシステムプレフィックス**（`[gateway]`/`[canvas]`/`[whatsapp/outbound]` 等、先頭の `gateway/`+`channels/` を畳んで末尾 2 セグメント保持）・サブシステム色（`NO_COLOR` 尊重）・`logRaw()`（QR/UX 用のプレフィックス無し出力）。
- **コンソールキャプチャ**：CLI は `console.log/...` を捕えてファイルログにも書きつつ stdout/stderr へも出す。

## 設定・使い方の要点

- **コンソールとファイルのレベルは別**：`logging.consoleLevel`（既定 `info`）・`logging.consoleStyle`（`pretty`/`compact`/`json`）は**ファイルログレベル（`logging.level`）を上げない**。verbose 限定詳細をファイルに残すなら `logging.level: debug|trace`。
- 例：`openclaw gateway --verbose --ws-log compact`（対応 req/res）／`--ws-log full`（フレーム完全）。
- WhatsApp メッセージ本文は `debug` でログ（表示は `--verbose`）。

## 注意点・落とし穴

- **リダクション**は OTLP ログレコードやセッショントランスクリプトのテキストシンクにも適用される。安全境界サーフェスは `redactSensitive: "off"` でも常に秘匿化（→ [[sources/logging]]）。
- `--verbose` でファイルログが詳細化すると誤解しないこと（コンソール/WS のみ）。

## 用語と略称

- **WS ログ** = Gateway WebSocket プロトコルの req/res ログ
- **サブシステムプレフィックス** = `[gateway]` 等のログ行頭ラベル
- **TTY 対応** = 端末なら色付き整形、非端末ならプレーン
- **JSONL** = 1 行 1 JSON のログ形式

## 関連ページ

- [[concepts/logging]] — 対応する概念ページ
- [[sources/logging]] — 利用者向けロギング概要
- [[components/gateway]] / [[concepts/observability]]
