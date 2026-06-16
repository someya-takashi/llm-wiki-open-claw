---
type: concept
aliases: [Command queue, コマンドキュー, queue, steering queue, ステアリングキュー]
tags: [queue, concurrency, steering, lanes, debounce]
related:
  - "[[concepts/agent-loop]]"
  - "[[concepts/session]]"
  - "[[concepts/messages]]"
  - "[[concepts/parallel-specialist-lanes]]"
sources:
  - "[[sources/concepts/queue]]"
  - "[[sources/concepts/queue-steering]]"
updated: 2026-06-14
---

# コマンドキュー

**コマンドキュー**は、全チャネルのインバウンド自動返信実行を**レーン対応 FIFO キュー**でシリアライズし、複数のエージェント実行の衝突を防ぎつつセッション間の安全な並列を残す仕組み。受信メッセージが実行中ターンをどう扱うか（**キューモード**：steer/followup/collect…）もここで決める。

## なぜ重要か

LLM 呼び出しは高コストで、近接して届く複数メッセージが共有リソース（セッションファイル・CLI stdin・上流レート制限）を奪い合うと壊れる。キューは [[concepts/session]] のセッションロックと組んで「**セッションごとアクティブ実行は 1 つ**」を保証し、`main`/`subagent`/`cron` などのレーンで全体並列度を制御する。既定の `steer` モードが「2 つ目の実行を起こさずアクティブターンの応答性を保つ」という UX を成立させており、[[concepts/agent-loop]] の直列化の実体になっている。

## キューモード（要点）

- **`steer`（既定）**：アクティブターンへ割り込み注入（ツール呼び出しは中断せず、次のモデル境界で配信）。
- **`followup`**：実行終了後の新ターンへ。**`collect`**：静穏ウィンドウ後に 1 ターンへ結合。**`steer-backlog`**：両方。**`queue`/`interrupt`**：レガシー。
- ランタイム別の配信タイミング（Pi の内部キュー vs Codex `turn/steer`、デバウンスの効き方）は [[sources/concepts/queue-steering]] を参照。
- 設定：`messages.queue`（`mode`/`debounceMs`/`cap`/`drop`、`byChannel`）、セッションごとは `/queue <mode>`。

詳細（レーンと同時実行上限・優先順位・診断）は [[sources/concepts/queue]]。並列ワークの設計論は [[concepts/parallel-specialist-lanes]]。

実行中ターンへのガイダンス注入（ステアリング）は [[sources/tools/steer]]（`/steer`）。

## 関連

- [[concepts/agent-loop]] / [[concepts/session]] / [[concepts/messages]] / [[concepts/retry]]
- [[sources/tools/steer]] / [[concepts/slash-commands]]
