---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/queue
source_path: raw/docs/concepts/queue.md
doc_section: concepts
title: "コマンドキュー"
ingested: 2026-06-14
tags: [queue, concurrency, steering, lanes, debounce, fifo]
related:
  - "[[concepts/queue]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/session]]"
---

# コマンドキュー（解説）

> 原典: `raw/docs/concepts/queue.md` ・ https://docs.openclaw.ai/ja-JP/concepts/queue

## 一言まとめ

全チャネルのインバウンド自動返信実行を、小さな**インプロセスのレーン対応 FIFO キュー**でシリアライズし、複数のエージェント実行が衝突するのを防ぎつつ、セッション間では安全な並列処理を残す仕組み。受信メッセージが実行中のターンをどう扱うか（ステアリング/フォローアップ）の**キューモード**もここで決める。

## 位置づけ

[[concepts/agent-loop]] が「セッションキーごとに直列化」されると述べた、その**直列化と並列制御の本体**。[[concepts/session]] のセッションロックと組み合わさり、[[concepts/messages]] パイプラインの「実行が走っている間に来たメッセージ」の扱いを定義する。並列ワークの設計論は [[concepts/parallel-specialist-lanes]]。

## 仕組み・ふるまい

### レーンと同時実行

- レーン対応 FIFO キューが各レーンを**設定可能な同時実行上限**でドレインする（未設定 1、`main` は 4、`subagent` は 8）。
- `runEmbeddedPiAgent` は **セッションキー**（レーン `session:<key>`）ごとにエンキュー → セッションごとにアクティブ実行は 1 つだけ。さらに**グローバルレーン**（既定 `main`）にキューされ、全体並列度は `agents.defaults.maxConcurrent` で制限。
- 追加レーン（`cron`/`cron-nested`/`nested`/`subagent`）でバックグラウンドジョブはインバウンド返信をブロックせず並列実行できる。
- タイピングインジケーターはエンキュー時にすぐ発火するので、順番待ち中も UX は変わらない。**外部依存やワーカースレッドは無く、純粋な TypeScript＋promises**。

### キューモード（受信が実行中ターンをどう扱うか）

- **`steer`（既定）**：アクティブランタイムにステアリングを注入。Pi は現在のアシスタントターンが**ツール呼び出しを終えた後・次の LLM 呼び出し前**に保留中のステアリングをまとめて配信。Codex app-server は 1 つの `turn/steer` を受ける。ステアリング不可ならフォローアップにフォールバック。
- **`queue`（レガシー）**：旧来の 1 件ずつステアリング。
- **`followup`**：現在の実行が終わった後のターン用に各メッセージをエンキュー。
- **`collect`**：静穏ウィンドウ後にキュー済みを**単一の**フォローアップターンへ結合（別チャネル/スレッド宛は個別ドレイン）。
- **`steer-backlog`**：今すぐステアリング**かつ**同じメッセージをフォローアップにも保持（ストリーミング面では重複に見えることあり）。
- **`interrupt`（レガシー）**：アクティブ実行を中止して最新メッセージを実行。

ランタイム別のタイミング詳細は [[sources/concepts/queue-steering]]、明示的な `/steer` は tools/steer。

## 設定・使い方の要点

- `messages.queue`（グローバル/`byChannel`）：既定 `mode: "steer"`・`debounceMs: 500`・`cap: 20`・`drop: "summarize"`。
- **キューオプション**（followup/collect/steer-backlog 等に適用）：`debounceMs`（ドレイン前の静穏ウィンドウ）/`cap`（セッションごと最大数）/`drop`（`summarize`＝最古を要約して残す・`old`＝要約なし削除・`new`＝満杯時に新規拒否）。
- **優先順位**：`/queue` オーバーライド → `byChannel` → `messages.queue.mode` → 既定 `steer`。セッションごとは `/queue <mode>`（`/queue collect debounce:0.5s cap:25` のように組み合わせ可、`/queue default` でクリア）。

## 注意点・落とし穴

- コマンドが止まって見える→詳細ログで `queued for...ms` 行を確認。
- 長時間 `processing` のまま進捗が無いセッションは診断（`diagnostics.stuckSessionWarnMs`）で `long_running`/`stalled`/`stuck` に分類され、`stuck` のみがレーンを解放してキュー済み作業をドレインできる。

## 用語と略称

- **FIFO** = First In, First Out（先入れ先出し）
- **レーン（lane）** = 同時実行上限を持つキューの筋道（`session:<key>`/`main`/`cron`/`subagent` 等）
- **ステアリング** = 実行中のターンへメッセージを割り込み注入すること
- **debounce** = 短時間の連続入力をまとめる静穏ウィンドウ

## 関連ページ

- [[concepts/queue]] — 対応する概念ページ
- [[sources/concepts/queue-steering]] — ステアリングのランタイム詳細（概念は [[concepts/queue]] に統合）
- [[concepts/agent-loop]] / [[concepts/session]] / [[concepts/messages]] / [[concepts/retry]]
