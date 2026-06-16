---
type: concept
aliases: [Compaction, コンパクション, 圧縮]
tags: [compaction, context, summary, overflow]
related:
  - "[[concepts/context]]"
  - "[[concepts/session-pruning]]"
  - "[[concepts/context-engine]]"
  - "[[concepts/memory]]"
  - "[[concepts/agent-loop]]"
sources:
  - "[[sources/concepts/compaction]]"
updated: 2026-06-14
---

# Compaction

**Compaction（コンパクション, 圧縮）**は、会話がコンテキストウィンドウの上限に近づいたとき古いメッセージを**要約へまとめ**てチャットを継続可能にする機構。要約はトランスクリプトに保存され、直近メッセージはそのまま残る――完全な履歴はディスクに残り、変わるのは次ターンでモデルに見える内容だけ。

## なぜ重要か

長い会話を「忘れずに畳む」ための土台。[[concepts/context]] をウィンドウ内に収める二大機構の片方で、もう片方の [[concepts/session-pruning]] とは**役割が違う**：Compaction は会話全体を要約して保存、プルーニングはツール結果をメモリ内のみトリム（保存しない）。[[concepts/agent-loop]] の途中で自動発火し、実装は [[concepts/context-engine]] が握る。Compaction の前に [[concepts/memory]] へ重要メモを保存する「メモリフラッシュ」が走るため、要約で情報を失いにくい。

## 押さえる点

- **自動**：上限接近時やプロバイダーのオーバーフローエラー時に発火（後者は Compaction して再試行）。**手動**：`/compact [指示]`。
- ツール呼び出しと `toolResult` はペアを保って分割される。
- 設定（`agents.defaults.compaction`）：`model`（要約専用モデル）・`truncateAfterCompaction`（後続トランスクリプト生成）・`notifyUser`・`memoryFlush.model`・プラグ可能 `provider`。
- クリーンに切りたいなら `/new`（Compaction せず新セッション）。

詳細・オーバーフローシグネチャ・全設定は [[sources/concepts/compaction]]。

## 関連

- [[concepts/context]] / [[concepts/session-pruning]] / [[concepts/context-engine]]
- [[concepts/memory]] / [[concepts/agent-loop]] / [[concepts/session]]
- 📝 ブログ（二次資料・外部）：不可逆圧縮を避け「復元可能な圧縮」「失敗の証拠を消さない」という原則 → [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]
- 📝 ブログ（二次資料・外部）：古い履歴を合成 user→assistant ペアへ要約し直近 N ターンを残す実装と要約プロンプト設計（OpenClaw の Compaction とほぼ同型）→ [[articles/context-engineering-session-memory]]
- 📝 ブログ（二次資料・Anthropic）：Compaction を長時間軸タスクの第一レバーと位置づけ、再現率→精度でプロンプト調整／ツール結果クリアを最軽量形とする総論 → [[articles/effective-context-engineering-for-ai-agents]]
- 📝 ブログ（二次資料・Anthropic）：「Compaction だけでは不十分＝次セッションへ完璧な指示を渡せない」ため進捗ファイル＋git で補う長時間ハーネス → [[articles/effective-harnesses-for-long-running-agents]]
