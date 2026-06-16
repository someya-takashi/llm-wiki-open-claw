---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/memory-lancedb
source_path: raw/docs/plugins/memory-lancedb.md
doc_section: plugins
title: "Memory LanceDB"
ingested: 2026-06-14
tags: [plugin, memory, lancedb, vector-db, embeddings, slot, active-memory]
related:
  - "[[concepts/memory]]"
  - "[[concepts/memory-search]]"
  - "[[components/plugin-system]]"
---

# Memory LanceDB（解説）

> 原典: `raw/docs/plugins/memory-lancedb.md` ・ https://docs.openclaw.ai/ja-JP/plugins/memory-lancedb

## 一言まとめ

長期記憶を **LanceDB（ローカルのベクトルデータベース）に保存し、埋め込みで想起**する同梱メモリ Plugin。モデルのターン前に関連記憶を自動想起し、応答後に重要事実を取り込む。

## 位置づけ

[[concepts/memory]] の差し替え可能バックエンドの一つ（既定 `memory-core`、他に QMD/Honcho）。検索は [[concepts/memory-search]] のハイブリッド検索系。Plugin スロット機構は [[components/plugin-system]]。

## 仕組み・ふるまい

- **Active Memory Plugin**：`plugins.slots.memory = "memory-lancedb"` でメモリスロットを占有（アクティブなメモリスロットを持てる Plugin は 1 つだけ。`memory-wiki` 等は併用可）。
- 埋め込みはプロバイダーベース（OpenAI 互換エンドポイント・Ollama 等）。想起/取り込みに制限あり、入力長がコンテキスト長を超える場合の対処やサポート外埋め込みモデルのトラブルシュートを規定。

## 設定・使い方の要点

- `plugins.slots.memory = "memory-lancedb"` ＋ 埋め込みプロバイダー設定。コマンドで記憶の検査/管理、ストレージは LanceDB ファイル。依存はランタイムローカル。
- OpenAI 互換 or Ollama の埋め込み例あり（[[providers/ollama]] 等）。

## 注意点・落とし穴

- 「Plugin は読み込まれるが記憶が表示されない」場合はスロット選択・埋め込み認証を確認。入力長超過は埋め込みモデルのコンテキスト上限に依存。

## 用語と略称

- **LanceDB** = 組み込み型のローカルベクトルデータベース
- **埋め込み（embedding）** = テキストを意味ベクトル化したもの（類似検索に使う）
- **Active Memory** = 応答前に先回りで想起する仕組み（[[concepts/active-memory]]）
- **スロット（slot）** = 排他的に 1 つ選ぶ Plugin カテゴリ

## 関連ページ

- [[concepts/memory]] / [[concepts/memory-search]] / [[concepts/active-memory]]
- [[sources/plugins/memory-wiki]] / [[components/plugin-system]]
