---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/agent-send
source_path: raw/docs/tools/agent-send.md
doc_section: tools
title: "エージェント送信（openclaw agent）"
ingested: 2026-06-14
tags: [agent-send, cli, single-turn, scripting, delivery]
related:
  - "[[components/cli]]"
  - "[[concepts/session]]"
  - "[[concepts/agent-runtimes]]"
---

# エージェント送信（openclaw agent）（解説）

> 原典: `raw/docs/tools/agent-send.md` ・ https://docs.openclaw.ai/ja-JP/tools/agent-send

## 一言まとめ

`openclaw agent` は、受信チャットメッセージなしに**コマンドラインから単一のエージェントターンを実行**する CLI。スクリプト化ワークフロー・テスト・プログラム的配信に使う。

## 位置づけ

[[components/cli]] のサブコマンド（対話 TUI とは別）。[[concepts/session]] を指定して 1 ターン回し、`--deliver` で実チャネルへ配信できる。ローカル/Gateway 経由のランタイムは [[concepts/agent-runtimes]]。

## 仕組み・ふるまい

- `openclaw agent --message "..."` で 1 ターン実行。`--session`・`--model`・`--deliver`（既定オフ）・`--local`（埋め込みランタイム）等のフラグ。Gateway 経由実行は [[concepts/tasks]] の `cli` タスクとして記録され得る。

## 設定・使い方の要点

- スクリプトから決定論的にエージェントを呼ぶ用途。配信したいときだけ `--deliver`。CLI バックエンド/Gateway モードに従う。

## 用語と略称

- **`openclaw agent`** = 単一ターンを実行する CLI コマンド
- **単一ターン（single turn）** = 1 回の受付→推論→返信
- **`--deliver`** = 結果を実チャネルへ配信するフラグ

## 関連ページ

- [[components/cli]] / [[concepts/session]] / [[concepts/agent-runtimes]]
- [[sources/web/tui]] / [[concepts/tasks]]
