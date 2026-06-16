---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/session
source_path: raw/docs/concepts/session.md
doc_section: concepts
title: "セッション管理"
ingested: 2026-06-14
tags: [session, routing, lifecycle, dm-scope, maintenance]
related:
  - "[[concepts/session]]"
  - "[[concepts/session-pruning]]"
  - "[[concepts/channel-docking]]"
---

# セッション管理（解説）

> 原典: `raw/docs/concepts/session.md` ・ https://docs.openclaw.ai/ja-JP/concepts/session

## 一言まとめ

OpenClaw は会話を **セッション**（連続した会話文脈の単位）に整理し、各メッセージを送信元（DM・グループ・Cron 等）に基づいてセッションへルーティングする。セッションのルーティング規則・ライフサイクル（リセット）・保存場所・メンテナンスを説明したページ。

## 位置づけ

[[concepts/agent-loop]] は **セッションキーごとに直列化**して実行され、その会話状態を保持する器が本ページのセッション。状態は [[components/gateway]] が所有し、トランスクリプトは [[concepts/agent]] が示した JSONL に残る。ツール結果のトリミングは [[concepts/session-pruning]]、長い履歴の要約は [[concepts/compaction]]、セッションを操作するエージェントツールは [[concepts/session-tool]]。

## 仕組み・ふるまい

### メッセージのルーティング

DM は既定で共有セッション、グループ/ルームはグループ・ルームごとに分離、Cron は実行ごとに新規、Webhook はフックごとに分離。

### DM 分離（重要なセキュリティ設定）

既定では**すべての DM が 1 セッションを共有**する（単一ユーザーなら問題ない）。⚠️ 複数人がエージェントに DM できる場合は **DM 分離を有効化しないと、Alice の私信が Bob に見えてしまう**。`session.dmScope` を設定する：`main`（既定・全 DM 共有）/ `per-peer`（送信者ごと、チャネル横断）/ `per-channel-peer`（チャネル＋送信者、**推奨**）/ `per-account-channel-peer`。同一人物が複数チャネルから来る場合は `session.identityLinks` で ID を束ねる。

### ライフサイクル（リセット）

- **日次リセット**（既定）：Gateway ホストのローカル時刻 04:00 に新セッション開始。鮮度は現在 `sessionId` の開始時刻基準。
- **アイドルリセット**（任意）：`session.reset.idleMinutes`。鮮度は最後の実ユーザー/チャネル操作基準（Heartbeat・Cron・exec はセッションを維持しない）。
- **手動**：`/new` または `/reset`（`/new <model>` はモデルも切替）。
- 両方設定時は先に切れた方が優先。プロバイダー所有のアクティブ CLI セッションは暗黙の日次では切られない。

### 保存場所

すべて Gateway が所有。ストア `~/.openclaw/agents/<agentId>/sessions/sessions.json`、トランスクリプト `.../<sessionId>.jsonl`。`sessions.json` は `sessionStartedAt`（日次リセット用）・`lastInteractionAt`（アイドル延長用）・`updatedAt`（一覧/刈り込み用、鮮度の信頼源ではない）を個別保持。

## 設定・使い方の要点

- **メンテナンス**（自動容量制限）：既定 `warn`、自動削除は `session.maintenance.mode: "enforce"`（`pruneAfter` `maxEntries`）。即時適用は `openclaw sessions cleanup --enforce`、プレビューは `--dry-run`。永続的な外部会話ポインターは保持し、合成 Cron/フック/Heartbeat/ACP/サブエージェントのエントリは経年削除できる。
- リンク済みチャネルへ返信先を移すには [[concepts/channel-docking]]（`/dock_*`）。
- 確認：`openclaw status`、`openclaw sessions --json`（`--active <分>`）、チャット内 `/status` `/context list`。`openclaw security audit` で DM スコープ等を検証。

## 注意点・落とし穴

- **マルチユーザーで `dmScope: main` のまま**は情報漏えいリスク。`per-channel-peer` 推奨。
- `dmScope` を `main` に戻した後の古い peer キー DM 行は `openclaw sessions cleanup --dry-run --fix-dm-scope` で整理（トランスクリプトは削除済みアーカイブとして保持）。

## 用語と略称

- **セッション** = 連続した会話文脈の単位（ルーティング・ライフサイクル・保存の対象）
- **DM** = Direct Message（個別チャット）
- **dmScope** = DM をどの粒度で分離するかの設定
- **JSONL** = JSON Lines（1 行 1 JSON のトランスクリプト形式）

## 関連ページ

- [[concepts/session]] — 対応する概念ページ
- [[concepts/session-pruning]] / [[concepts/session-tool]] / [[concepts/channel-docking]]
- [[concepts/agent-loop]] / [[concepts/compaction]] / [[concepts/multi-agent]]
