---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/memory
source_path: raw/docs/concepts/memory.md
doc_section: concepts
title: "メモリの概要"
ingested: 2026-06-14
tags: [memory, markdown, MEMORY.md, dreaming, backend, tools]
related:
  - "[[concepts/memory]]"
  - "[[concepts/agent-workspace]]"
  - "[[concepts/memory-search]]"
---

# メモリの概要（解説）

> 原典: `raw/docs/concepts/memory.md` ・ https://docs.openclaw.ai/ja-JP/concepts/memory

## 一言まとめ

OpenClaw のメモリは、エージェントのワークスペースに書き込まれる**プレーンな Markdown ファイル**で実現される――モデルが「記憶」するのはディスク上の内容だけで、隠れた状態は無い。どのファイルに何を置くか、検索ツール、バックエンド、自動統合（Dreaming・メモリフラッシュ）を俯瞰するハブページ。

## 位置づけ

メモリは「会話を越えて持続する知識」で、コンテキスト（[[concepts/context]]）が「今ウィンドウに載っているもの」なのと対比される。ファイルは [[concepts/agent-workspace]] に置かれ、`MEMORY.md` は [[concepts/system-prompt]] が毎ターン注入する。検索は [[concepts/memory-search]]、長期昇格は [[concepts/dreaming]]、短期フォローアップは [[concepts/commitments]]、応答前の先回り想起は [[concepts/active-memory]] が担う。

## 仕組み・ふるまい

### 3 つのメモリファイル

- **`MEMORY.md`** — 長期メモリ。永続的な事実・好み・判断の**精選された層**。メインの非公開 DM セッション開始時に読み込まれる（共有/グループでは読まない）。生ログやアーカイブにはしない。
- **`memory/YYYY-MM-DD.md`** — 日次ノートの**作業層**。`memory_search`/`memory_get` 用に索引化されるが、毎ターン注入はされない。今日＋昨日は自動読み込み。
- **`DREAMS.md`**（任意）— Dream Diary と Dreaming スイープの概要（→ [[concepts/dreaming]]）。

時間とともにエージェントは日次ノートから有用な内容を `MEMORY.md` へ抽出し、古い項目を削除する（生成済みワークスペース指示や Heartbeat が定期実行できる）。`MEMORY.md` がブートストラップ予算を超えると、ディスクは保持しつつ注入コピーは切り詰められる。

### メモリツール

`memory_search`（セマンティック検索で関連ノートを発見）と `memory_get`（特定ファイル/行範囲の読み取り）。どちらもアクティブなメモリ Plugin（既定 `memory-core`）が提供する。

### バックエンド（差し替え可能）

- **組み込み（既定）**: SQLite ベース。キーワード/ベクトル/ハイブリッド検索を備え追加依存なし（→ [[sources/concepts/memory-builtin]]）。
- **QMD**: 再ランキング・クエリ拡張・ワークスペース外索引化を備えたローカル優先サイドカー（→ [[sources/concepts/memory-qmd]]）。
- **Honcho**: ユーザーモデリング・クロスセッションメモリを持つ AI ネイティブな専用サービス（Plugin）（→ [[sources/concepts/memory-honcho]]）。
- **LanceDB**: バンドル済みの LanceDB バックエンド（Plugin、自動想起/キャプチャ）。
- 別軸として **Memory Wiki** Plugin は、永続知識を主張・証拠・鮮度追跡を備えた「Wiki Vault」へコンパイルする（`wiki_search`/`wiki_get`/`wiki_apply`/`wiki_lint`）。アクティブなメモリ Plugin を置き換えず、来歴の豊かな知識層を横に足す。

### 自動メモリフラッシュ

[[concepts/compaction]] が会話を要約する前に、OpenClaw は重要なコンテキストをメモリファイルへ保存させる**サイレントターン**を実行する（既定有効）。`agents.defaults.compaction.memoryFlush.model` でこの保守ターンだけローカルモデルに固定できる。

## 設定・使い方の要点

- 記憶させたいときは普通に頼めばよい（「TypeScript を好むと覚えて」）。エージェントが適切なファイルへ書く。
- 埋め込みプロバイダー（OpenAI/Gemini/Voyage/Mistral）の API キーがあれば `memory_search` は**ハイブリッド検索**で自動的に動く。
- CLI：`openclaw memory status`（索引状態）/`search "query"`/`index --force`。`/context list`・`/context detail`・`openclaw doctor` で raw/注入サイズと切り詰め状態を確認。
- 根拠付きバックフィル：`openclaw memory rem-backfill --path ./memory --stage-short-term` で履歴ノートを再生し短期 Dreaming ストアへステージング（`--rollback` で取り消し）。`MEMORY.md` はディープ昇格でのみ書かれる。

## 注意点・落とし穴

- `MEMORY.md` は**精選された長期要約**に保つ。肥大すると注入が切り詰められる――詳細は `memory/*.md` へ移すか予算を上げる。
- 永続的でないフォローアップ（「面接後に確認」）は `MEMORY.md` ではなく [[concepts/commitments]] が扱う。明示的リマインダーは cron（automation/cron-jobs）。

## 用語と略称

- **MEMORY.md / memory/YYYY-MM-DD.md / DREAMS.md** = 長期・日次・夢日記のメモリファイル
- **memory-core** = 既定のアクティブメモリ Plugin
- **メモリフラッシュ** = Compaction 前に重要コンテキストを保存させるサイレントターン
- **ハイブリッド検索** = ベクトル類似度＋キーワードを組み合わせた検索

## 関連ページ

- [[concepts/memory]] — 対応する概念ページ
- [[concepts/memory-search]] / [[concepts/active-memory]] / [[concepts/dreaming]] / [[concepts/commitments]]
- [[sources/concepts/memory-builtin]] / [[sources/concepts/memory-qmd]] / [[sources/concepts/memory-honcho]]
- [[concepts/agent-workspace]] / [[concepts/compaction]]
