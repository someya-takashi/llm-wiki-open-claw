---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/queue-steering
source_path: raw/docs/concepts/queue-steering.md
doc_section: concepts
title: "ステアリングキュー"
ingested: 2026-06-14
tags: [queue, steering, runtime, pi, codex, debounce]
related:
  - "[[concepts/queue]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/agent-runtimes]]"
---

# ステアリングキュー（解説）

> 原典: `raw/docs/concepts/queue-steering.md` ・ https://docs.openclaw.ai/ja-JP/concepts/queue-steering

## 一言まとめ

実行中のセッションがすでにストリーミングしている間にメッセージが届いたとき、別実行を起こす代わりに**アクティブなランタイムへ送る（ステアリングする）**仕組みの、ランタイム別の配信タイミング詳細。公開モードは [[concepts/queue]] と共通だが、Pi とネイティブ Codex app-server で実装が異なる。

## 位置づけ

[[concepts/queue]] のキューモード（特に `steer`）を、[[concepts/agent-runtimes]] の実行バックエンド視点で掘り下げたページ。`steer` がなぜ「ツール呼び出しを中断しない」のか、Pi の内部キューと Codex の `turn/steer` の違いが分かる。

## 仕組み・ふるまい

### ランタイム境界（ステアリングは実行中ツールを中断しない）

Pi はモデル境界でステアリングを確認する：①アシスタントがツール呼び出しを要求 → ②Pi がツールバッチを実行 → ③ターン終了イベント → ④キュー済みステアリングを排出 → ⑤次の LLM 呼び出し前にユーザーメッセージとして追加。これでツール結果は要求元アシスタントメッセージと対応し続け、次のモデル呼び出しが最新入力を見られる。ネイティブ Codex app-server は内部キューの代わりに `turn/steer` を公開し、OpenClaw が同じモードを適用する。Codex レビューと手動 Compaction ターンは同一ターンのステアリングを拒否する。

### モード別（アクティブ実行の動作）

| モード | アクティブ実行 | フォローアップ |
| --- | --- | --- |
| `steer`（既定） | 次の境界で保留中ステアリングをまとめて注入 | 不可時のみフォールバック |
| `queue`（レガシー） | 境界ごとに 1 件ずつ注入（Codex は個別 `turn/steer`） | 不可時のみフォールバック |
| `steer-backlog` | `steer` と同じ | 同じメッセージを後続にも保持 |
| `followup` | ステアリングしない | 後で実行 |
| `collect` | ステアリングしない | デバウンス後に 1 ターンへ結合 |
| `interrupt` | アクティブ実行を中止して最新を開始 | なし |

### デバウンス

`messages.queue.debounceMs` は collect/followup/steer-backlog と `steer` のフォローアップフォールバックに適用。**Pi のアクティブ `steer` 自体はデバウンスタイマーを使わない**（次のモデル境界まで自然にまとまるため）。ネイティブ Codex はバッチ `turn/steer` 送信前の静穏ウィンドウとして同じ値を使う。

## 設定・使い方の要点

- バースト例（ツール実行中に 4 人が送信）：`steer` は次のモデル判断前に 4 件を到着順で受ける、`queue` は 1 件ずつ、`collect` はアクティブ実行終了後にデバウンスして 1 ターンへ。
- 1 件ずつの旧動作が要るときだけ `queue`、結合してドロップポリシーを効かせたいなら `collect`。

## 注意点・落とし穴

- ステアリングは常に現在のアクティブ実行を対象にし、新セッション作成・ツールポリシー変更・送信者分割はしない（受信プロンプトに送信者/ルート情報が含まれるので、次のモデル呼び出しは各送信者を識別できる）。

## 用語と略称

- **ステアリング（steering）** = 実行中ターンへの割り込みメッセージ注入
- **モデル境界** = 1 回の LLM 呼び出しの区切り（ツールバッチ実行後）
- **turn/steer** = Codex app-server のステアリング API

## 関連ページ

- [[concepts/queue]] — 対応する概念ページ（キューモード全体）
- [[concepts/agent-loop]] / [[concepts/agent-runtimes]] / [[concepts/messages]]
