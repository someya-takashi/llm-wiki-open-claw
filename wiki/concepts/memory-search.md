---
type: concept
aliases: [Memory search, メモリ検索, hybrid search]
tags: [memory-search, embedding, vector, bm25, hybrid]
related:
  - "[[concepts/memory]]"
  - "[[concepts/active-memory]]"
  - "[[concepts/context]]"
sources:
  - "[[sources/concepts/memory-search]]"
updated: 2026-06-14
---

# メモリ検索

**メモリ検索**（`memory_search`）は、表現が元テキストと違っても関連ノートを見つける仕組み。メモリを小さなチャンクに索引化し、**埋め込み（ベクトル）と キーワード（BM25）の 2 経路を並列実行してマージする「ハイブリッド検索」**で精度と再現率を両立する。

## なぜ重要か

[[concepts/memory]] が Markdown ファイルとして持つ知識を「使える」ものにする中核。ベクトル検索が意味の近さ（「gateway host」≈「the machine running OpenClaw」）を、BM25 が完全一致（ID・エラー文字列・設定キー）を担い、片方しか使えなくても字句ランキングへ縮退する頑健さがある。[[concepts/active-memory]] の先回り想起も、[[concepts/dreaming]] の昇格シグナルも、このパイプラインに乗る。

## 押さえる点

- 埋め込みプロバイダー（OpenAI/Gemini/Voyage/Mistral/Bedrock/Local/Ollama）。OpenAI/Gemini/Voyage/Mistral のキーがあれば**自動検出**で有効。決定的にするには明示固定（実行後のランタイム失敗は自動フォールバックしない）。
- 任意の品質改善：`temporalDecay`（時間減衰、既定半減期 30 日、常緑ファイルは減衰しない）と `mmr`（多様性、ほぼ重複を減らす）。
- マルチモーダル（Gemini Embedding 2 で画像/音声索引化）、実験的なセッションメモリ検索。

検索フロー図・プロバイダー表・トラブルシュートは [[sources/concepts/memory-search]]。

## 関連

- [[concepts/memory]] / [[concepts/active-memory]]
- [[sources/concepts/memory-builtin]] / [[sources/concepts/memory-qmd]]
