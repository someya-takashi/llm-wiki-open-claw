---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/memory-qmd
source_path: raw/docs/concepts/memory-qmd.md
doc_section: concepts
title: "QMD メモリエンジン"
ingested: 2026-06-14
tags: [memory, backend, qmd, sidecar, reranking, local]
related:
  - "[[concepts/memory]]"
  - "[[concepts/memory-search]]"
---

# QMD メモリエンジン（解説）

> 原典: `raw/docs/concepts/memory-qmd.md` ・ https://docs.openclaw.ai/ja-JP/concepts/memory-qmd

## 一言まとめ

QMD は OpenClaw と並行して動く**ローカル優先の検索サイドカー**（別プロセス）。BM25・ベクトル検索・再ランキングを単一バイナリにまとめ、ワークスペースのメモリファイル以外も索引化できる。組み込みより高再現率だが外部バイナリが要る。

## 位置づけ

[[concepts/memory]] のバックエンドの 1 つ。検索の基本は [[concepts/memory-search]] と同じだが、**再ランキング・クエリ拡張・ワークスペース外索引化・セッション文字起こし索引化**を上積みする。QMD が使えないときは組み込みエンジンへ自動フォールバックする。

## 仕組み・ふるまい

- **組み込みより追加されること**：再ランキング＋クエリ拡張（高再現率）／追加ディレクトリ索引化（プロジェクトドキュメント等）／セッション文字起こし索引化（過去会話の想起）／API キー不要の完全ローカル（`node-llama-cpp` で GGUF を自動 DL）／自動フォールバック。
- **サイドカー**：OpenClaw はワークスペースのメモリと `memory.qmd.paths` からコレクションを作り、QMD マネージャーが定期（既定 5 分）に `qmd update`（セマンティックモードでは `qmd embed`）を別サブプロセスで実行。Gateway 起動では既定で QMD を初期化せず、最初に使われるまで遅延（`memory.qmd.update.startup` で `idle`/`immediate`）。
- 検索モードは `searchMode`（既定 `search`＝BM25 のみ、`vsearch`/`query` も）。失敗時は `qmd query` で再試行。完全失敗時は組み込み SQLite にフォールバックし、再試行嵐を避けるためバックオフ。

## 設定・使い方の要点

- 前提：`npm i -g @tobilu/qmd`、拡張許可 SQLite（macOS は `brew install sqlite`）、QMD が Gateway の `PATH` 上。Windows は WSL2 推奨。
- 有効化：`memory.backend: "qmd"`。`~/.openclaw/agents/<id>/qmd/` に自己完結ホームを作り自動管理。
- 追加パス：`memory.qmd.paths: [{ name, path, pattern }]`（結果は `qmd/<collection>/<path>`、`memory_get` が解釈）。セッション索引化：`memory.qmd.sessions.enabled: true`。
- スコープ：既定でダイレクト＋チャネル（グループ除く）。`memory.qmd.scope` で変更。モデル上書きは `QMD_EMBED_MODEL`/`QMD_RERANK_MODEL` 等の環境変数。

## 注意点・落とし穴

- 初回検索は遅い（再ランキング/クエリ拡張用 GGUF を約 2GB 自動 DL）。`qmd query "test"` で事前ウォーム。
- `spawn qmd ENOENT`：Gateway の `PATH` が対話シェルと違う。`memory.qmd.command` で絶対パス固定。
- BM25 のみ運用で llama.cpp ビルドが走る→`searchMode: "search"`。タイムアウト→`memory.qmd.limits.timeoutMs`（既定 4000ms）。

## 用語と略称

- **サイドカー（sidecar）** = 本体と並走する補助プロセス
- **再ランキング（reranking）** = 一次検索結果を別モデルで並べ替えて精度を上げる
- **クエリ拡張** = 検索語を補って再現率を上げる
- **GGUF** = ローカルモデルのファイル形式

## 関連ページ

- [[concepts/memory]] / [[concepts/memory-search]]
- [[sources/concepts/memory-builtin]] / [[sources/concepts/memory-honcho]]
