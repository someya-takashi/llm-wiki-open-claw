---
type: concept
aliases: [Session, セッション, session management]
tags: [session, routing, lifecycle, dm-scope]
related:
  - "[[concepts/session-pruning]]"
  - "[[concepts/session-tool]]"
  - "[[concepts/channel-docking]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/multi-agent]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/session]]"
updated: 2026-06-14
---

# セッション

**セッション**は OpenClaw の会話単位――各メッセージは送信元（DM・グループ・ルーム・Cron・Webhook）に基づいてセッションへルーティングされ、状態は [[components/gateway]] が所有する。[[concepts/agent-loop]] は**セッションキーごとに直列化**して走るため、セッションは「会話文脈の器」であると同時に「並行制御の単位」でもある。

## なぜ重要か

セッションの粒度設定はそのまま**プライバシー境界**になる。既定では全 DM が 1 セッションを共有するため、複数人がエージェントに DM できる環境で `session.dmScope` を分離しないと、ある人の私信が別の人に見えてしまう。`per-channel-peer`（チャネル＋送信者）が推奨で、同一人物の複数チャネルは `identityLinks` で束ねる。ライフサイクル（日次/アイドル/手動リセット）とメンテナンス（容量制限）が、コンテキストの鮮度とストレージのバランスを取る。

## 押さえる点

- **保存場所**：Gateway 所有。`sessions.json`（`sessionStartedAt`=日次リセット基準 / `lastInteractionAt`=アイドル延長 / `updatedAt`）＋ `<sessionId>.jsonl` トランスクリプト。
- **リセット**：日次（既定 04:00）・アイドル（`reset.idleMinutes`）・手動（`/new` `/reset`）。
- 周辺：ツール結果の刈り込みは [[concepts/session-pruning]]、会話要約は [[concepts/compaction]]、操作ツールは [[concepts/session-tool]]、返信先の付け替えは [[concepts/channel-docking]]、複数エージェント分離は [[concepts/multi-agent]]。

詳細・dmScope の全オプション・メンテナンス設定は [[sources/concepts/session]]。

## 関連

- [[concepts/session-pruning]] / [[concepts/session-tool]] / [[concepts/channel-docking]]
- [[concepts/agent-loop]] / [[concepts/compaction]]
- 📝 ブログ（二次資料・外部）：Session をメモリオブジェクトとして短期メモリ（トリミング/要約）を管理する OpenAI Cookbook 解説 → [[articles/context-engineering-session-memory]]
- 📝 ブログ（二次資料・Anthropic）：離散セッション（前任の記憶なし＝シフト制）をまたいで作業を橋渡しする長時間エージェントハーネス → [[articles/effective-harnesses-for-long-running-agents]]
