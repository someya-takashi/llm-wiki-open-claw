---
type: concept
aliases: [Context engine, コンテキストエンジン]
tags: [context-engine, plugin, compaction, lifecycle, slot]
related:
  - "[[concepts/context]]"
  - "[[concepts/compaction]]"
  - "[[concepts/memory]]"
sources:
  - "[[sources/concepts/context-engine]]"
updated: 2026-06-14
---

# コンテキストエンジン

**コンテキストエンジン**は、各実行のモデルコンテキストを「どう組み立て・どう要約し・サブエージェント境界でどう管理するか」を制御する**差し替え可能な実装スロット**（`plugins.slots.contextEngine`）。組み込み `legacy` が既定で、必要なときだけ Plugin エンジンを選ぶ。

## なぜ重要か

[[concepts/context]] が「何を送るか」の概念だとすると、本ページは**それを実装として握る拡張点**。4 つのライフサイクル（取り込み → アセンブル → コンパクト化 → ターン後）にエンジンが関与し、`assemble()` がトークン予算内のメッセージと任意の `systemPromptAddition` を返す。これにより、ベクトル検索や DAG 要約のような独自のコンテキスト戦略を、コア本体を触らず差し込める。

## 押さえる点

- **`ownsCompaction`**：`true` ならエンジンが [[concepts/compaction]] を所有し Pi 自動 Compaction を無効化。`false`（既定）は委譲モードで、`delegateCompactionToRuntime(...)` を呼ぶ。
- ⚠️ `false` でも **legacy へ自動フォールバックしない**。no-op の `compact()` は危険。
- 登録失敗・id 未解決時は**実行が失敗**する（自動フォールバックなし）。直すか `"legacy"` に戻す。
- メモリ Plugin（[[concepts/memory]]、`plugins.slots.memory`）とは別物だが連携できる。

導入手順・インターフェース・所有/委譲モードは [[sources/concepts/context-engine]]。

## 関連

- [[concepts/context]] / [[concepts/compaction]] / [[concepts/memory]]
- 📝 ブログ（二次資料・外部）：コンテキスト組み立て戦略（KV キャッシュ・ツール可用性・外部メモリ）の一般論 → [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]
