---
type: concept
aliases: [Parallel specialist lanes, 並列の専門レーン, lanes]
tags: [parallel, lanes, multi-agent, concurrency, coordinator]
related:
  - "[[concepts/multi-agent]]"
  - "[[concepts/queue]]"
  - "[[concepts/session]]"
  - "[[concepts/session-tool]]"
sources:
  - "[[sources/concepts/parallel-specialist-lanes]]"
updated: 2026-06-14
---

# 並列の専門レーン

**並列の専門レーン**は、1 つの Gateway が複数チャット/ルームを別々のエージェントへ振り分けつつ UX を高速に保つための**設計論**。並列性を「エージェントを増やすこと」ではなく、**希少リソースへの競合を減らす設計問題**として扱う。

## なぜ重要か

[[concepts/multi-agent]] で頭脳を分けても、[[concepts/session]] のセッションロック・グローバルなモデル容量・ツール容量・コンテキスト予算という**共有ボトルネック**は残る。専門レーンはその上に「どのエージェントが何を所有し、何をチャットに残し、何をバックグラウンドへ回すか」のポリシーを足す――並列化が効くのは**実ボトルネックへの競合を減らすときだけ**、という原則が中核。OpenClaw 既存の [[concepts/queue]]（グローバル並列制限）と組み合わせて働く。

## 押さえる点（推奨ロールアウト）

1. **レーン契約＋重い作業のバックグラウンド化**（最安・最大効果）：各レーンに Owns / Does not own / Chat budget / Handoff / Tool posture を書面化。
2. **優先度と同時実行制御**：`maxConcurrent`・`subagents.maxConcurrent`/`delegationMode`・`messages.queue` を価値に合わせて調整。
3. **コーディネーター**：複数レーン稼働後に所有者追跡・重複検出・引き継ぎルーティングの調整役を追加（**ここから始めない**）。

フェーズ別設定・最小レーン契約テンプレは [[sources/concepts/parallel-specialist-lanes]]。

## 関連

- [[concepts/multi-agent]] / [[concepts/queue]] / [[concepts/session-tool]]
