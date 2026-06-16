---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/commitments
source_path: raw/docs/concepts/commitments.md
doc_section: concepts
title: "推定されたコミットメント"
ingested: 2026-06-14
tags: [commitments, memory, follow-up, heartbeat, inferred]
related:
  - "[[concepts/commitments]]"
  - "[[concepts/memory]]"
  - "[[concepts/session]]"
---

# 推定されたコミットメント（解説）

> 原典: `raw/docs/concepts/commitments.md` ・ https://docs.openclaw.ai/ja-JP/concepts/commitments

## 一言まとめ

コミットメントは**短期間だけ保持されるフォローアップ用メモリ**。会話から「あとで確認すべき機会」が生まれたことに OpenClaw が気づき、期限が来たら Heartbeat で自然なチェックインを届ける。`MEMORY.md` のような永続事実でも正確なリマインダーでもない、**メモリと自動化の中間**に位置する。**オプトイン（既定オフ）**。

## 位置づけ

[[concepts/memory]] の長期事実とは別レーンの「会話に紐づく短期義務」。明示的リマインダー（「3 時に教えて」）は cron（automation/cron-jobs）の領分で、コミットメントは**推論された**フォローアップ専用。配信は Heartbeat（gateway/heartbeat）に乗り、[[concepts/session]]・チャネルにスコープされる。

## 仕組み・ふるまい

- 例：「明日面接がある」→後で確認。「徹夜した」→後で眠れたか尋ねる。「変わったらフォローする」と言った→その未完了ループを追跡。
- エージェントの返信後、OpenClaw は**別コンテキストの隠れたバックグラウンド抽出パス**を走らせ、推論されたフォローアップだけを探す（表示会話には書かず、メインエージェントには抽出を意識させない）。
- 高信頼候補が見つかると、エージェント ID・セッションキー・元チャネルと配信先・期限ウィンドウ・短いチェックイン案・（指示でない）メタデータを保存する。
- 配信：期限が来ると Heartbeat が同じエージェント/チャネルスコープの Heartbeat ターンにコミットメントを足す。モデルは自然なチェックインを 1 件送るか `HEARTBEAT_OK` で破棄。`target: "none"` の Heartbeat なら内部に留まり外部送信しない。元の会話テキストは再生されず、配信ターンは OpenClaw ツール無しで実行される。
- 期限は**作成後少なくとも 1 Heartbeat 間隔以降**に制限される（推論した瞬間にそのまま返ってこない）。

## 設定・使い方の要点

- 有効化：`openclaw config set commitments.enabled true`、`commitments.maxPerDay 3`（既定 3、エージェントセッションごとのローリング 1 日内の配信上限）。
- 管理 CLI：`openclaw commitments`（一覧）／`--all`／`--agent main`／`--status snoozed`／`dismiss cm_abc123`。
- スコープは作成時の**正確なエージェント＋チャネル**に固定される（別エージェント/別チャネルからは配信されない）。これは「グローバルなリマインダー」ではなく「同じ会話が続いている感覚」を作るための設計。

## 注意点・落とし穴

- **プライバシーとコスト**：抽出は LLM パスを使うため、有効にすると対象ターン後にバックグラウンドのモデル使用量が増える。隠れているが、フォローアップ判断のため直近のやり取りを読むことがある。保存は運用メモリ（長期メモリではない）。
- フォローアップが出ない→`commitments.enabled` 確認、`openclaw commitments --all` で保留/破棄/スヌーズ/期限切れを確認、Heartbeat が動いているか、`maxPerDay` に達していないか、明示リマインダーは cron 側に出るべき点を確認。

## 用語と略称

- **コミットメント** = 会話から推論された短期フォローアップメモリ
- **推論された（inferred）** = ユーザーが明示的に頼まなくても会話から導かれた
- **Heartbeat** = 定期実行でチェックイン等を届ける仕組み
- **スヌーズ（snooze）** = 配信を先送りした状態

## 関連ページ

- [[concepts/commitments]] — 対応する概念ページ
- [[concepts/memory]] / [[concepts/active-memory]] / [[concepts/session]]
