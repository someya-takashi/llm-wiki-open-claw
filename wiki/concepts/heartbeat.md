---
type: concept
aliases: [Heartbeat, ハートビート, HEARTBEAT_OK]
tags: [heartbeat, agent-turn, schedule, automation]
related:
  - "[[concepts/session]]"
  - "[[concepts/commitments]]"
  - "[[concepts/dreaming]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/heartbeat]]"
updated: 2026-06-14
---

# Heartbeat

**Heartbeat** は、エージェントの**メインセッションで定期的に走るエージェントターン**。モデルが「注意を要するもの」（受信箱・カレンダー・期限の来たフォローアップ）を、通知を乱発せずに表面化するための「軽い定期チェックイン」。既定間隔は `30m`（Anthropic OAuth 検出時 `1h`）。

## なぜ重要か

OpenClaw を「呼ばれたら答える」だけでなく**自発的に動くアシスタント**にする土台。スケジュール実行という点では cron に似るが、Heartbeat は**メインセッションのターン**であり、バックグラウンドタスクレコードを作らない（明示的リマインダーは cron、推論フォローアップは [[concepts/commitments]]、長期記憶の昇格配信は [[concepts/dreaming]]――いずれも期限が来ると Heartbeat ターンに乗る）。応答契約 **`HEARTBEAT_OK`**（注意不要のサイン）が「静かなときは黙る」を実現する。

## 押さえる点

- 設定（`agents.defaults.heartbeat` / `agents.list[].heartbeat`）：`every`（`0m` で無効）・`target`（`none` 既定/`last`/`<channel>`）・`activeHours`・`isolatedSession`/`lightContext`（コスト削減）。いずれかのエージェントに `heartbeat` ブロックがあると**そのエージェントのみ**実行。
- **`HEARTBEAT.md`**（任意の常設チェックリスト、`tasks:` ブロックで interval 別確認）。空や期限タスク無しならスキップ（`reason=empty-heartbeat-file`/`no-tasks-due`）。
- Cron 作業中は自動延期。表示は `showOk`/`showAlerts`/`useIndicator`（全 false なら実行自体スキップ）。
- ⚠️ **コスト**：完全なターンなので短い間隔はトークンを食う。`HEARTBEAT.md` にシークレットを入れない。

詳細（応答契約・配信ルーティング・手動起動・推論配信）は [[sources/gateway/heartbeat]]。

## 関連

- [[concepts/automation]]（自動化の全体像。Cron との対比） / [[concepts/cron]]
- [[concepts/session]] / [[concepts/commitments]] / [[concepts/dreaming]]
- [[components/gateway]] / [[sources/gateway/config-agents]]
