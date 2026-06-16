---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/memory-builtin
source_path: raw/docs/concepts/memory-builtin.md
doc_section: concepts
title: "組み込みメモリエンジン"
ingested: 2026-06-14
tags: [memory, backend, sqlite, fts5, bm25, embedding, default]
related:
  - "[[concepts/memory]]"
  - "[[concepts/memory-search]]"
---

# 組み込みメモリエンジン（解説）

> 原典: `raw/docs/concepts/memory-builtin.md` ・ https://docs.openclaw.ai/ja-JP/concepts/memory-builtin

## 一言まとめ

組み込みエンジンは**既定のメモリバックエンド**。メモリインデックスをエージェントごとの SQLite データベースに保存し、追加依存なしでそのまま動く。ほとんどのユーザーに適した選択肢。

## 位置づけ

[[concepts/memory]] のバックエンドの 1 つ（既定）で、検索の考え方は [[concepts/memory-search]] と共通。再ランキング/クエリ拡張/ワークスペース外索引化が要れば QMD（[[sources/concepts/memory-qmd]]）、自動ユーザーモデリングが要れば Honcho（[[sources/concepts/memory-honcho]]）へ。

## 仕組み・ふるまい

- **キーワード検索**：FTS5 全文インデックス（BM25 スコアリング）。
- **ベクトル検索**：対応プロバイダーの埋め込み。
- **ハイブリッド検索**：両者を組み合わせて最良の結果。
- **CJK 対応**：中国語・日本語・韓国語向け trigram トークン化。
- **sqlite-vec アクセラレーション**（任意）：DB 内ベクトルクエリ。読み込めなければ自動でプロセス内コサイン類似度にフォールバック。
- 索引：`MEMORY.md` と `memory/*.md` を約 400 トークン（80 トークン重複）でチャンク化し `~/.openclaw/memory/<agentId>.sqlite` へ。ファイル変更で 1.5 秒デバウンスの再索引、プロバイダー/モデル/チャンク設定変更で全再構築。`openclaw memory index --force` で手動。

## 設定・使い方の要点

- 自動検出：OpenAI/Gemini/Voyage/Mistral/DeepInfra のキーがあればベクトル検索が有効（設定不要）。明示は `memorySearch.provider`。
- ローカル埋め込み強制：`node-llama-cpp` ランタイムを隣に入れ、`provider: "local"`・`local.modelPath` を GGUF に向ける。
- ワークスペース外の Markdown は `memorySearch.extraPaths` で索引化。

## 注意点・落とし穴

- 埋め込みプロバイダーが無いとキーワード検索のみ。
- `Vector store: unavailable`（sqlite-vec の読み込み問題）と `Embeddings: unavailable`（プロバイダー/認証/モデル準備）は別物。`openclaw memory status --deep` が両者を別々に報告する。

## 用語と略称

- **FTS5** = SQLite の全文検索拡張（BM25 ランキング）
- **sqlite-vec** = SQLite 内でベクトル検索を高速化する拡張
- **trigram** = 3 文字単位のトークン化（CJK の分かち書き対策）
- **GGUF** = ローカルモデルのファイル形式

## 関連ページ

- [[concepts/memory]] / [[concepts/memory-search]]
- [[sources/concepts/memory-qmd]] / [[sources/concepts/memory-honcho]]
