---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/automation/hooks
source_path: raw/docs/automation/hooks.md
doc_section: automation
title: "フック（内部フック）"
ingested: 2026-06-14
tags: [hooks, hook-md, lifecycle-events, automation, scripts]
related:
  - "[[concepts/hooks]]"
  - "[[concepts/automation]]"
  - "[[sources/plugins/hooks]]"
---

# フック（内部フック）（解説）

> 原典: `raw/docs/automation/hooks.md` ・ https://docs.openclaw.ai/ja-JP/automation/hooks

## 一言まとめ

オペレーターが `HOOK.md`＋ハンドラを置いて、**ライフサイクル/メッセージイベントに反応するスクリプト**を走らせる仕組み。`/new`・Compaction 前後・Gateway 起動・メッセージ受信などのタイミングで発火する。

## 位置づけ

[[concepts/hooks]] の中核ソース。Plugin がコードでインプロセス介入する **Plugin SDK フック（[[sources/plugins/hooks]]）とは別物**——こちらはオペレーターの軽量スクリプト。[[concepts/automation]] の「イベント駆動」担当。

## 仕組み・ふるまい

- **イベント**：`command:new`/`reset`/`stop`/`command`、`session:compact:before`/`after`、`session:patch`、`agent:bootstrap`、`gateway:startup`/`shutdown`/`pre-restart`、`message:received`/`transcribed`/`preprocessed`/`sent`。
- **HOOK.md 形式**：`emoji`/`events`（待受配列）/`export`/`os`/`requires`（bins/anyBins/env/config）/`always`/`install`。ハンドラ実装でイベントコンテキストを受け取る。
- **検出**：フックパックとして配置、バンドルフック（`session-memory`＝`/new`/`/reset` でセッションを `<workspace>/memory/` に保存、`bootstrap-extra-files`＝`agent:bootstrap` で追加ファイル注入）。

## 設定・使い方の要点

- フックパックを workspace/グローバルに置き、`requires` で環境ゲート。`agent:bootstrap` は [[concepts/agent-workspace]] のブートストラップ注入前に走る。

## 注意点・落とし穴

- ⚠️ Plugin の SDK フック（`before_tool_call` 等、[[sources/plugins/hooks]]）と混同しない。内部フックは「`HOOK.md` スクリプト」、Plugin フックは「コードのインプロセス介入」。

## 用語と略称

- **HOOK.md** = フックのメタデータ＋手順ファイル
- **フックパック** = `HOOK.md`＋ハンドラのディレクトリ
- **ライフサイクルイベント** = `/new`・Compaction・Gateway 起動等の節目
- **`agent:bootstrap`** = ワークスペース起動ファイル注入前のイベント

## 関連ページ

- [[concepts/hooks]] — 対応する概念ページ
- [[concepts/automation]] / [[sources/plugins/hooks]] / [[concepts/agent-workspace]]
- [[concepts/compaction]] / [[concepts/memory]]
