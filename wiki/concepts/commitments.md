---
type: concept
aliases: [Commitments, コミットメント, inferred commitments]
tags: [commitments, memory, follow-up, heartbeat]
related:
  - "[[concepts/memory]]"
  - "[[concepts/session]]"
  - "[[concepts/dreaming]]"
  - "[[concepts/heartbeat]]"
sources:
  - "[[sources/concepts/commitments]]"
updated: 2026-06-14
---

# 推定されたコミットメント

**コミットメント**は、会話から「あとで確認すべき機会」が生まれたことに OpenClaw が気づき、期限が来たら Heartbeat で自然なチェックインを届ける、**短期のフォローアップメモリ**。`MEMORY.md` のような永続事実でも、正確なリマインダーでもない――**メモリと自動化の中間**。**オプトイン（既定オフ）**。

## なぜ重要か

「明日面接がある」「徹夜した」のような発言から、ユーザーが明示的に頼まなくても**自然な follow-up** を生む。これは [[concepts/memory]] の長期事実とは別レーンで、明示的リマインダー（cron）とも区別される。スコープが作成時の**正確なエージェント＋チャネル**（[[concepts/session]]）に固定されるのが肝で、「グローバルなリマインダー」ではなく「同じ会話が続いている感覚」を作る。

## 押さえる点

- 抽出は返信後の**隠れたバックグラウンド LLM パス**（表示会話に書かず、メインエージェントに意識させない）。高信頼候補のみ保存。
- 配信は Heartbeat 経由で、期限は作成後 1 Heartbeat 間隔以降に制限（推論した瞬間に返ってこない）。モデルはチェックインを送るか `HEARTBEAT_OK` で破棄。
- 有効化：`commitments.enabled true`・`maxPerDay`（既定 3）。管理：`openclaw commitments [--all|--agent|dismiss <id>]`。
- コスト：有効にすると対象ターン後にバックグラウンドのモデル使用量が増える。

詳細・スコープ・トラブルシュートは [[sources/concepts/commitments]]。

## 関連

- [[concepts/automation]]（自動化の中での位置づけ） / [[concepts/heartbeat]]
- [[concepts/memory]] / [[concepts/session]] / [[concepts/active-memory]]
