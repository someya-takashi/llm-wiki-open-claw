---
type: concept
aliases: [Hooks, フック, 内部フック, HOOK.md]
tags: [hooks, hook-md, lifecycle-events, automation, event-driven]
related:
  - "[[concepts/automation]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/agent-workspace]]"
components:
  - "[[components/gateway]]"
  - "[[components/plugin-system]]"
sources:
  - "[[sources/automation/hooks]]"
updated: 2026-06-14
---

# フック（Hooks）

フック（hooks, ライフサイクルやメッセージのイベントに反応して実行されるスクリプト/コールバック）は、OpenClaw の**イベント駆動の自動化**。OpenClaw には性格の違う 2 系統がある：

| 系統 | 何か | ソース |
|---|---|---|
| **内部フック** | オペレーターの `HOOK.md`＋ハンドラ。`/new`・Compaction・Gateway 起動・メッセージ受信などで発火 | [[sources/automation/hooks]] |
| **Plugin フック** | Plugin がコードでツール呼び出し等にインプロセス介入（`before_tool_call`/`llm_input` 等） | [[sources/plugins/hooks]] |

⚠️ この 2 つは別物。本ページは主に**内部フック**（自動化としての `HOOK.md` スクリプト）を扱う。

## なぜ重要か

「セッションがリセットされたらメモリを保存する」「ブートストラップ前に追加ファイルを注入する」のような**ライフサイクルの節目に挟む処理**は、cron や heartbeat（時刻駆動）では表現できない。フックは「このイベントが起きたら、これを実行」をオペレーターが宣言でき、[[concepts/automation]] のイベント駆動の柱になる。バンドルの `session-memory` フックは [[concepts/memory]] への退避を、`bootstrap-extra-files` は [[concepts/agent-workspace]] への注入を担う。

## 仕組み（要点）

- **イベント**：`command:new`/`reset`/`stop`、`session:compact:before`/`after`、`session:patch`、`agent:bootstrap`、`gateway:startup`/`shutdown`/`pre-restart`、`message:received`/`transcribed`/`preprocessed`/`sent`。
- **HOOK.md**：`events`（待受）/`requires`（環境ゲート）/`os`/`always`/`install`。フックパックとして workspace/グローバルに配置。詳細は [[sources/automation/hooks]]。

## 既存 wiki とのつながり

内部フックは [[concepts/agent-loop]] とメッセージ処理（[[concepts/messages]]）の節目に挟まり、[[components/gateway]] が発火する。一方の Plugin フック（[[sources/plugins/hooks]]）は [[components/plugin-system]] の SDK 面で、より低レベルにツール/モデルへ介入する——用途で使い分ける。

## 代表ソース

- [[sources/automation/hooks]] — 内部フックのイベント・HOOK.md 形式・バンドルフック

## 関連ページ

- [[concepts/automation]] / [[concepts/agent-loop]] / [[concepts/agent-workspace]]
- [[sources/plugins/hooks]]（Plugin SDK フック・別系統） / [[components/gateway]] / [[components/plugin-system]]
