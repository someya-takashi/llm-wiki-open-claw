---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/btw
source_path: raw/docs/tools/btw.md
doc_section: tools
title: "BTW（ちなみに補足の質問）"
ingested: 2026-06-14
tags: [btw, side-question, slash-command, session, codex]
related:
  - "[[concepts/slash-commands]]"
  - "[[concepts/session]]"
  - "[[concepts/session-tool]]"
---

# BTW（ちなみに補足の質問）（解説）

> 原典: `raw/docs/tools/btw.md` ・ https://docs.openclaw.ai/ja-JP/tools/btw

## 一言まとめ

`/btw`（エイリアス `/side`）は、**現在のセッションについての短い横道の質問**を、通常の会話履歴を変えずに尋ねる仕組み。Claude Code の `/btw` を OpenClaw の Gateway/マルチチャネルに合わせたもの。

## 位置づけ

[[concepts/slash-commands]] のコマンドで、[[concepts/session]] のトランスクリプトに書き込まずに脇道の確認をする。[[concepts/session-tool]] の補助的なセッション操作。

## 仕組み・ふるまい

- 現在のセッションを背景コンテキストに使い、将来のセッションコンテキストは変えない・トランスクリプトに書かない・ライブのサイド結果として配信。
- Codex ハーネスセッションでは一時的な Codex サイドスレッドとして実行（現在の権限/ネイティブツールを使用）、Codex 以外は従来の直接ワンショット。

## 設定・使い方の要点

- メインタスクを進めたまま「今何してる?」のような確認をしたいときに使う（`/btw <question>`）。

## 用語と略称

- **BTW** = By The Way（ちなみに）の横道質問
- **`/side`** = `/btw` のエイリアス
- **サイドスレッド** = 本流を汚さない一時的な実行
- **ワンショット** = 履歴に残さない単発呼び出し

## 関連ページ

- [[concepts/slash-commands]] / [[concepts/session]] / [[concepts/session-tool]]
- [[sources/plugins/codex-harness]]
