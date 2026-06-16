---
type: concept
aliases: [Context, コンテキスト, context window]
tags: [context, context-window, tokens, compaction, pruning]
related:
  - "[[concepts/system-prompt]]"
  - "[[concepts/context-engine]]"
  - "[[concepts/compaction]]"
  - "[[concepts/agent-loop]]"
sources:
  - "[[sources/concepts/context]]"
updated: 2026-06-14
---

# コンテキスト

**コンテキスト**＝1 回の実行で OpenClaw がモデルへ送る**すべて**：[[concepts/system-prompt]]＋会話履歴＋ツール呼び出し/結果＋添付。モデルの**コンテキストウィンドウ（トークン上限）**に収める必要がある。

## なぜ重要か

エージェントのコスト・速度・賢さは「ウィンドウに何を載せるか」で決まる。OpenClaw はこれを可視化（`/context list|detail|map`、`/status`、`/usage tokens`）し、空ける手段（`/compact`、剪定）を持つ。とくに **ツールは 2 重コスト**（プロンプト内のツールリスト＋送信されるツールスキーマ JSON）を持ち、見えないスキーマがウィンドウを食う点が実務上の落とし穴。

## 押さえる点（保持・圧縮・剪定）

- **通常履歴**：ポリシーで圧縮/剪定されるまでトランスクリプトに保持。
- **[[concepts/compaction]]**：要約を残し最近のメッセージは維持。
- **剪定（pruning）**：メモリ内プロンプトからのみ古いツール結果を落とす（トランスクリプトは不変）。
- **コンテキスト ≠ メモリ**：メモリはディスクに保存・再読込でき、コンテキストは今ウィンドウに載っているもの。

何を組み立て要約するかの実装は [[concepts/context-engine]] が制御する。詳細・`/context` の出力例は [[sources/concepts/context]]。

## 関連

- [[concepts/system-prompt]] / [[concepts/context-engine]] / [[concepts/compaction]]
- [[concepts/agent-loop]] / [[concepts/session]]
- 📝 ブログ（二次資料・外部）：KV キャッシュ中心設計・append-only・few-shot ドリフトなど context 設計の一般原則 → [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]
- 📝 ブログ（二次資料・外部）：長い会話でウィンドウを破綻させない短期メモリ管理（トリミング vs 要約）の対比と評価 → [[articles/context-engineering-session-memory]]
- 📝 ブログ（二次資料・Anthropic）：context rot／注意の予算という理論的背景と「最小の高シグナル集合」という総論 → [[articles/effective-context-engineering-for-ai-agents]]
