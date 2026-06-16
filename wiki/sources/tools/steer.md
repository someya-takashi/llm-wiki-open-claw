---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/steer
source_path: raw/docs/tools/steer.md
doc_section: tools
title: "Steer（誘導）"
ingested: 2026-06-14
tags: [steer, steering, active-run, queue, guidance]
related:
  - "[[concepts/queue]]"
  - "[[concepts/messages]]"
  - "[[concepts/multi-agent]]"
---

# Steer（誘導）（解説）

> 原典: `raw/docs/tools/steer.md` ・ https://docs.openclaw.ai/ja-JP/tools/steer

## 一言まとめ

`/steer`（エイリアス `/tell`）は、**すでにアクティブな実行へガイダンスを注入**する仕組み。新しいターンを開始するためでなく、「処理中のこの実行を途中で調整する」ためのもの。

## 位置づけ

[[concepts/queue]] のステアリング機構（キューモードと独立に動く）。[[concepts/messages]] の割り込み配信で、サブエージェント（[[concepts/multi-agent]]）や ACP セッション（[[concepts/acp]]）にも送れる。

## 仕組み・ふるまい

- **現在のセッション**：アクティブ実行にガイダンスを注入。アイドル時は新実行を開始しない。
- **ステアリングとキュー**：`/queue` モードとは独立。配信タイミングはランタイム別（Pi/Codex）。
- **サブエージェント**：`/subagents steer <id> <message>` で実行中サブエージェントを誘導。**ACP セッション**にも誘導可能。

## 設定・使い方の要点

- 長い実行の途中で方針を変えたい/補足したいときに使う（`/steer <message>`）。

## 用語と略称

- **Steer（誘導）** = 実行中のターンにガイダンスを注入すること
- **ステアリング** = 実行を途中で調整する仕組み（≠ 新ターン開始）
- **キューモード** = 受信メッセージの処理戦略（steer/queue/followup 等）

## 関連ページ

- [[concepts/queue]] / [[concepts/messages]] / [[concepts/multi-agent]]
- [[concepts/acp]] / [[sources/tools/subagents]] / [[concepts/slash-commands]]
