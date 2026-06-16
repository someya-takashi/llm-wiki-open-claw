---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/heartbeat
source_path: raw/docs/gateway/heartbeat.md
doc_section: gateway
title: "Heartbeat"
ingested: 2026-06-14
tags: [heartbeat, agent-turn, schedule, HEARTBEAT.md, automation]
related:
  - "[[concepts/heartbeat]]"
  - "[[concepts/session]]"
  - "[[concepts/commitments]]"
---

# Heartbeat（解説）

> 原典: `raw/docs/gateway/heartbeat.md` ・ https://docs.openclaw.ai/ja-JP/gateway/heartbeat

## 一言まとめ

Heartbeat は**メインセッションで定期的なエージェントターンを実行**し、モデルが「注意を要するもの」を通知を乱発せずに表面化できるようにする仕組み。スケジュールされたメインセッションのターンであり、バックグラウンドタスクレコードは作らない。

## 位置づけ

[[concepts/heartbeat]] の本体。[[concepts/session]] のメインセッションで走り、期限が来た [[concepts/commitments]]（推論フォローアップ）や [[concepts/dreaming]] の配信もこの Heartbeat ターンに乗る。明示的なスケジュール実行は cron（automation/cron-jobs）で、Heartbeat は「軽い定期チェックイン」という棲み分け。

## 仕組み・ふるまい

- **既定間隔 `30m`**（Anthropic OAuth/トークン認証〔Claude CLI 再利用含む〕検出時は `1h`）。`agents.defaults.heartbeat.every`／`agents.list[].heartbeat.every`、`0m` で無効。
- **既定プロンプト**（そのままユーザーメッセージとして送られる）：「`HEARTBEAT.md` があれば読み、厳密に従う。古いタスクを推測/反復しない。注意が要らなければ `HEARTBEAT_OK` と返す」。
- **応答契約**：注意不要なら **`HEARTBEAT_OK`**（先頭/末尾なら確認応答として扱われ、残りが `≤ ackMaxChars`〔既定 300〕なら破棄）。アラート時は `HEARTBEAT_OK` を含めず本文のみ。ツール対応実行は `heartbeat_respond`（`notify` フラグ）で構造化応答も可。
- **延期**：Cron 作業がアクティブ/キュー中は自動延期。`skipWhenBusy: true` で自分のサブエージェント/ネストレーンでも延期（他エージェントのビジーでは延期しない）。

## 設定・使い方の要点

- 主な設定（`agents.defaults.heartbeat` / `agents.list[].heartbeat`）：`every`・`model`（上書き）・`target`（`none`〔既定〕/`last`/`<channel id>`）・`to`/`accountId`・`directPolicy`（`allow`/`block`）・`activeHours`（営業時間制限、IANA tz）・`isolatedSession`（毎回新セッション＝履歴なし）・`lightContext`（`HEARTBEAT.md` のみ注入）・`prompt`（上書き）・`ackMaxChars`。
- **スコープと優先順位**：いずれかの `agents.list[]` に `heartbeat` ブロックがあると、**そのエージェントのみ**が Heartbeat を実行。表示は `channels.<ch>.heartbeat`（`showOk`/`showAlerts`/`useIndicator`）で制御（3 つ全 false なら実行自体スキップ）。
- **`HEARTBEAT.md`**（任意）：小さな常設チェックリスト。空（見出しのみ）なら `reason=empty-heartbeat-file` でスキップ。`tasks:` ブロック（`name`/`interval`/`prompt`）で interval ごとの確認を入れられ、期限が来たタスクが無ければ `reason=no-tasks-due` でスキップ。
- **手動起動**：`openclaw system event --text "..." --mode now`（即時）/`--mode next-heartbeat`。

## 注意点・落とし穴

- **コスト意識**：完全なエージェントターンなので短い間隔はトークンを食う。`isolatedSession: true`（約 100K→2–5K トークン）＋`lightContext: true`＋安価な `model`＋小さい `HEARTBEAT.md`＋`target: "none"` で軽量化。
- ⚠️ `HEARTBEAT.md` に**シークレットを入れない**（プロンプトコンテキストに入る）。
- `activeHours` の `start==end` は幅ゼロ＝常にスキップ。24/7 は `activeHours` 省略か `00:00`–`24:00`。
- 小さいローカルモデルで Heartbeat 実行後、メインターンでコンテキストオーバーフローする場合はセッションのランタイムモデルをプライマリへ戻す。

## 用語と略称

- **Heartbeat** = 定期的に走るメインセッションのエージェントターン
- **`HEARTBEAT_OK`** = 「注意不要」を表す確認応答トークン
- **HEARTBEAT.md** = ワークスペースの Heartbeat チェックリスト
- **isolatedSession / lightContext** = 履歴なし新セッション / `HEARTBEAT.md` のみ注入
- **activeHours** = Heartbeat を許す時間帯

## 関連ページ

- [[concepts/heartbeat]] — 対応する概念ページ
- [[concepts/session]] / [[concepts/commitments]] / [[concepts/dreaming]]
- [[sources/gateway/config-agents]] — `agents.*.heartbeat` の設定面
