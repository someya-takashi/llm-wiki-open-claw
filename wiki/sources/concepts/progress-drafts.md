---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/progress-drafts
source_path: raw/docs/concepts/progress-drafts.md
doc_section: concepts
title: "進行中の下書き"
ingested: 2026-06-14
tags: [progress-drafts, streaming, preview, tool-progress, ux]
related:
  - "[[concepts/progress-drafts]]"
  - "[[concepts/streaming]]"
  - "[[concepts/messages]]"
---

# 進行中の下書き（解説）

> 原典: `raw/docs/concepts/progress-drafts.md` ・ https://docs.openclaw.ai/ja-JP/concepts/progress-drafts

## 一言まとめ

Progress drafts（進行中の下書き）は、長時間のエージェントターンを「一時的なステータス返信の山」にせず、**1 つの作業中メッセージ**を更新し続けてチャット上で動いているように見せる仕組み。完了時にその下書きを最終回答に変換する。

## 位置づけ

[[concepts/streaming]] のプレビューストリーミング `progress` モードの中身。回答テキストをトークン単位で見せるより「**いま何が起きているか**」を見せたいツール多用ターン向け。配線は [[concepts/messages]]、ツール進捗行の書式は streaming と共有。

## 仕組み・ふるまい

- 有効化すると、ターンが実際に作業を始めた後に**1 つの作業中メッセージ**を作り、読み取り/計画/ツール呼び出し/承認待ちの間それを更新、安全なら最終回答に変換する。
- **2 部構成**：①ラベル（`Thinking...` `Shelling...` 等の短い開始/ステータス行）②進捗行（`🛠️ Bash: run tests`、`🔎 Web Search: ...` のようなコンパクトな実行更新）。ラベルは意味ある作業が始まり 5 秒ビジー or 2 つ目の作業イベント後に表示。プレーンテキストのみの返信では出ない。
- 既定ラベルは `auto`（カニ/海をモチーフにした 1 語ラベルプール: `Shelling...` `Clawing...` `Molting...` …）。固定ラベル・独自プール・非表示（`label: false`）も可。
- 進捗行は実行イベント（ツール開始・計画・承認・コマンド出力・パッチ要約）から生成。既定は `/verbose` と同じ `toolProgressDetail: "explain"`（簡潔）、`"raw"` で生コマンドも（ノイズ増）。`maxLines` で表示行数制限、長い行は自動切り詰め。Slack は `progress.render: "rich"` で Block Kit 表示。

## 設定・使い方の要点

- 有効化：`channels.<channel>.streaming.mode: "progress"`（通常はこれだけ）。
- モード選択：`off`（最終のみ）/`partial`（回答テキストが進捗）/`block`（大きめチャンクのプレビュー）/`progress`（ツール多用・長時間ターン）。
- ラベルは `streaming.progress.label`（`auto`/固定/`false`、`labels` で独自プール）、ツール行は `progress.toolProgress`（`false` で単一下書き維持しつつ行を隠す）。
- チャネル別トランスポート：Discord/Telegram/Matrix は送信→編集、Slack はネイティブストリーム or ドラフト投稿、Teams はネイティブ進捗、Mattermost はドラフト投稿。安全な編集が無いチャネルは入力インジケーターか最終のみにフォールバック。

## 注意点・落とし穴

- **最終回答しか出ない**→そのチャネル/アカウントで `streaming.mode: "progress"` か確認。グループ/引用返信では安全に編集できず下書きが無効になることがある。
- **編集ではなく新しい最終メッセージが出る**→安全のためのフォールバック（メディア返信・長い回答・明示的な返信先・古い Telegram 下書き・Slack スレッド欠落・削除済みプレビュー・ネイティブ最終化失敗）。テキストを失うより新規送信を選ぶ意図的な設計。
- Teams は他チャネルと挙動が違う（個人チャットのネイティブストリーム、`block` は Teams ブロック配信）。

## 用語と略称

- **Progress draft（進行中の下書き）** = 更新され続ける単一の作業中メッセージ
- **ラベル / 進捗行** = 短いステータス語 / コンパクトな実行更新行
- **最終化（finalize）** = 下書きを最終回答に変換する処理
- **Block Kit** = Slack の構造化リッチ表示形式

## 関連ページ

- [[concepts/progress-drafts]] — 対応する概念ページ
- [[concepts/streaming]] / [[concepts/messages]]
