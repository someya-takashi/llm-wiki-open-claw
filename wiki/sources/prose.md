---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/prose
source_path: raw/docs/prose.md
doc_section: root
title: "OpenProse"
ingested: 2026-06-14
tags: [openprose, prose, workflow, multi-agent, dsl, plugin, slash-command]
related:
  - "[[components/plugin-system]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/parallel-specialist-lanes]]"
---

# OpenProse（解説）

> 原典: `raw/docs/prose.md` ・ https://docs.openclaw.ai/ja-JP/prose
>
> ℹ️ docs ルート直下の独立ページ。OpenProse は OpenClaw 専用機能ではなく、ポータブルな Markdown ファーストのワークフローフォーマット（`www.prose.md`）で、OpenClaw には **Plugin として**載る。

## 一言まとめ

OpenProse は AI セッションをオーケストレーションする `.prose` ワークフロー DSL。OpenClaw では `open-prose` Plugin＋`/prose` スラッシュコマンドとして提供され、**明示的な制御フローで複数のサブエージェントを起動**できる。

## 位置づけ

[[components/plugin-system]] のオプトイン Plugin（既定無効）。`.prose` プログラムは [[concepts/multi-agent]] のサブエージェント生成（`sessions_spawn`）と [[concepts/parallel-specialist-lanes]] の並列実行を、再現可能なファイルとして書き下す手段。決定論的・承認ゲート付きワークフローの Lobster とは対比関係。

## 仕組み・ふるまい

- `.prose` ファイルに `agent`（model/prompt）と `parallel`/`session` ブロックを書き、明示的並列で調査→統合などを実行。OpenProse の概念は OpenClaw プリミティブにマップ：セッション起動→`sessions_spawn`、ファイル→`read`/`write`、Web 取得→`web_fetch`（[[concepts/session-tool]]）。
- 状態は workspace の `.prose/`（`runs/<timestamp>/` に program/state/bindings/agents）。state モード：filesystem（既定）/in-context/sqlite/postgres（実験）。
- `/prose run <file|handle/slug|URL>`・`compile`・`examples`・`update`。リモートは `https://p.prose.md/<handle>/<slug>` に解決。

## 設定・使い方の要点

- 有効化：`openclaw plugins enable open-prose` → Gateway 再起動（or ローカル checkout を `plugins install`）。
- ツール許可リストが `sessions_spawn`/`read`/`write`/`web_fetch` をブロックすると `.prose` は失敗（[[sources/tools/skills-config]]）。

## 注意点・落とし穴

- ⚠️ **`.prose` ファイルはコードとして扱う**（実行前に確認、ツール許可リスト＋承認ゲートで副作用を制御）。postgres 認証情報はサブエージェントログに流れるため最小権限の専用 DB を使う。

## 用語と略称

- **OpenProse / `.prose`** = Markdown ファーストのワークフロー DSL とそのファイル
- **DSL** = Domain-Specific Language（特定用途の言語）
- **VM** = Virtual Machine（OpenProse の命令実行系）
- **`sessions_spawn`** = サブエージェントを起動する OpenClaw ツール

## 関連ページ

- [[components/plugin-system]] / [[concepts/multi-agent]] / [[concepts/parallel-specialist-lanes]]
- [[concepts/session-tool]] / [[concepts/slash-commands]] / [[concepts/skills]]
