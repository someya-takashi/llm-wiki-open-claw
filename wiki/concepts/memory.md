---
type: concept
aliases: [Memory, メモリ, MEMORY.md]
tags: [memory, markdown, backend, recall]
related:
  - "[[concepts/memory-search]]"
  - "[[concepts/active-memory]]"
  - "[[concepts/dreaming]]"
  - "[[concepts/commitments]]"
  - "[[concepts/compaction]]"
  - "[[concepts/agent-workspace]]"
sources:
  - "[[sources/concepts/memory]]"
  - "[[sources/concepts/memory-builtin]]"
  - "[[sources/concepts/memory-qmd]]"
  - "[[sources/concepts/memory-honcho]]"
updated: 2026-06-14
---

# メモリ

OpenClaw のメモリは、エージェントのワークスペースに置かれた**プレーンな Markdown ファイル**で実現される――`MEMORY.md`（精選された長期メモリ）・`memory/YYYY-MM-DD.md`（日次の作業ノート）・任意の `DREAMS.md`。**モデルが記憶するのはディスク上の内容だけで、隠れた状態は無い**という透明性が中核。

## なぜ重要か

「会話を越えて持続する知識」を、ブラックボックスのベクトル DB ではなく**人間が読み書きできるファイル**に置くことで、検査・編集・バックアップ（[[concepts/agent-workspace]] の git）が効く。コンテキスト（[[concepts/context]]）が「今ウィンドウに載っているもの」なのに対し、メモリは「ディスクに残し後で呼び出すもの」。`MEMORY.md` は [[concepts/system-prompt]] が毎ターン注入し、日次ノートは [[concepts/memory-search]] でオンデマンド取得される。

## 構成

- **検索**：[[concepts/memory-search]]（ハイブリッド = ベクトル＋BM25）。ツールは `memory_search`/`memory_get`。
- **先回り想起**：[[concepts/active-memory]]（応答前に関連メモリを表面化）。
- **長期昇格**：[[concepts/dreaming]]（短期シグナル→`MEMORY.md`）。
- **短期フォローアップ**：[[concepts/commitments]]（会話に紐づく義務）。
- **Compaction 連携**：[[concepts/compaction]] の前に重要メモを保存する「メモリフラッシュ」が走る。

## 代表的なバックエンド（差し替え可能 / `plugins.slots.memory`）

- **組み込み（既定）**：SQLite ベース、追加依存なし（→ [[sources/concepts/memory-builtin]]）。
- **QMD**：再ランキング・ワークスペース外索引化のローカル優先サイドカー（→ [[sources/concepts/memory-qmd]]）。
- **Honcho**：自動ユーザーモデリングの AI ネイティブ専用サービス（→ [[sources/concepts/memory-honcho]]）。
- **LanceDB**：ローカルのベクトル DB に保存し埋め込みで自動想起する Active Memory Plugin（→ [[sources/plugins/memory-lancedb]]）。
- **Memory Wiki**：永続メモリを来歴付きの知識 Vault（wiki）へコンパイルする同梱 Plugin。Active Memory を置き換えず横に並ぶ（→ [[sources/plugins/memory-wiki]]）。

ファイルレイアウト・ツール・根拠付きバックフィルの詳細は [[sources/concepts/memory]]。

## 関連

- [[concepts/memory-search]] / [[concepts/active-memory]] / [[concepts/dreaming]] / [[concepts/commitments]]
- [[concepts/agent-workspace]] / [[concepts/context]] / [[concepts/compaction]]
- 📝 ブログ（二次資料・外部）：ファイルシステムを無制限・永続・可逆な外部メモリとして使う設計論 → [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]
- 📝 ブログ（二次資料・外部）：状態ベース長期メモリの蒸留→統合→注入ループ（global/session スコープ分離・忘却・PII ガードレール）→ [[articles/context-engineering-personalization]]
- 📝 ブログ（二次資料・Anthropic）：構造化ノート取り（agentic memory・NOTES.md・file-based memory tool）と just-in-time 検索の総論 → [[articles/effective-context-engineering-for-ai-agents]]
- 📝 ブログ（二次資料・Anthropic）：進捗ファイルにセッション要約を残し次のコンテキストウィンドウで読み返す長時間タスクのハーネス実践 → [[articles/effective-harnesses-for-long-running-agents]]
