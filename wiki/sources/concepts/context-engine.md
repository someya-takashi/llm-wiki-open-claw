---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/context-engine
source_path: raw/docs/concepts/context-engine.md
doc_section: concepts
title: "コンテキストエンジン"
ingested: 2026-06-14
tags: [context-engine, plugin, compaction, lifecycle, memory]
related:
  - "[[concepts/context-engine]]"
  - "[[concepts/context]]"
  - "[[concepts/compaction]]"
---

# コンテキストエンジン（解説）

> 原典: `raw/docs/concepts/context-engine.md` ・ https://docs.openclaw.ai/ja-JP/concepts/context-engine

## 一言まとめ

**コンテキストエンジン**は、各実行のモデルコンテキストを「どう組み立て、古い履歴をどう要約し、サブエージェント境界をまたいでどう管理するか」を制御する差し替え可能な部品。OpenClaw には組み込み `legacy` が同梱され既定で使われ、別の挙動が要るときだけ Plugin エンジンを入れて選ぶ、という仕組みを説明したページ。

## 位置づけ

[[concepts/context]] が「モデルに送る全体」だとすると、本ページはそれを**実際に組み立てる実装スロット**。[[concepts/compaction]]（履歴圧縮）はこのエンジンの責務の 1 つで、メモリ Plugin（`plugins.slots.memory`）とは別物だが連携できる。`plugins.slots.contextEngine` という Plugin スロットで 1 つだけ排他選択される。

## 仕組み・ふるまい

### 4 つのライフサイクルポイント

モデルプロンプトを実行するたびにエンジンが関与する：①**取り込み（ingest）**＝新メッセージを自前のデータストアへ保存/索引化、②**アセンブル（assemble）**＝各実行前にトークン予算内の順序付きメッセージ（＋任意の `systemPromptAddition`）を返す、③**コンパクト化（compact）**＝ウィンドウが埋まったか `/compact` 時に古い履歴を要約、④**ターン後（afterTurn）**＝状態永続化やバックグラウンド Compaction、索引更新。Codex ハーネスでは、組み立て済みコンテキストを Codex の開発者指示＋ターンプロンプトに投影して同じライフサイクルを適用する（Codex はネイティブの履歴とコンパクターを所有）。

### legacy エンジン

OpenClaw の元の挙動を維持：取り込み＝no-op、アセンブル＝パススルー（既存の sanitize→validate→limit パイプライン）、コンパクト化＝組み込み要約 Compaction に委譲、ターン後＝no-op。ツールも `systemPromptAddition` も提供しない。

### Plugin エンジンと `ownsCompaction`

`api.registerContextEngine(id, factory)` で登録。`info`・`ingest`・`assemble`（`AssembleResult` を返す）・`compact` が必須、`bootstrap`/`ingestBatch`/`afterTurn`/`prepareSubagentSpawn`/`onSubagentEnded`/`dispose` が任意。`ownsCompaction:true` ならエンジンが Compaction を所有し Pi 組み込み自動 Compaction を無効化、`false`（既定）なら Pi の自動 Compaction が走りつつ `compact()` は `/compact`・オーバーフロー回復で呼ばれる。有効なパターンは 2 つ：**所有モード**（自前アルゴリズム＋`ownsCompaction:true`）と**委譲モード**（`ownsCompaction:false`＋`compact()` から `delegateCompactionToRuntime(...)` を呼ぶ）。

## 設定・使い方の要点

- 確認：`openclaw doctor`、または `jq '.plugins.slots.contextEngine' ~/.openclaw/openclaw.json`。
- 導入：`openclaw plugins install <pkg>`（ローカルは `-l ./dir`）→ `plugins.slots.contextEngine: "<id>"` と `plugins.entries.<id>.enabled: true` → Gateway 再起動。legacy に戻すにはキー削除か `"legacy"`。
- スロットは実行時に排他（解決されるエンジンは 1 つ）。選択中 Plugin をアンインストールするとスロットは既定（legacy）に戻る（`plugins.slots.memory` も同様）。

## 注意点・落とし穴

- ⚠️ `ownsCompaction:false` でも **legacy へ自動フォールバックしない**。no-op の `compact()` は非所有エンジンには危険（そのスロットの `/compact`・オーバーフロー回復を無効化してしまう）。
- エンジン登録失敗や id 未解決時、OpenClaw は**自動フォールバックせず実行が失敗する**。Plugin を直すか `"legacy"` に戻すまで動かない。
- メモリ Plugin はエンジンと別。Active Memory プロンプトを使うエンジンは `buildMemorySystemPromptAddition(...)` を優先利用する。剪定はどのエンジンでも引き続き走る。

## 用語と略称

- **アセンブル** = 実行前にトークン予算内のメッセージ集合を組み立てる工程
- **ownsCompaction** = エンジンが Compaction を所有するか（true で Pi 自動 Compaction を無効化）
- **スロット（slot）** = 実行時に 1 実装だけを排他選択する Plugin の差し込み口
- **DAG** = Directed Acyclic Graph（有向非巡回グラフ。要約戦略の一例）

## 関連ページ

- [[concepts/context-engine]] — 対応する概念ページ
- [[concepts/context]] / [[concepts/compaction]] / [[concepts/memory]]
