---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/parallel-specialist-lanes
source_path: raw/docs/concepts/parallel-specialist-lanes.md
doc_section: concepts
title: "並列の専門レーン"
ingested: 2026-06-14
tags: [parallel, lanes, multi-agent, concurrency, queue, coordinator]
related:
  - "[[concepts/parallel-specialist-lanes]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/queue]]"
---

# 並列の専門レーン（解説）

> 原典: `raw/docs/concepts/parallel-specialist-lanes.md` ・ https://docs.openclaw.ai/ja-JP/concepts/parallel-specialist-lanes

## 一言まとめ

1 つの Gateway が複数チャット/ルームを別々のエージェントへ振り分けつつ UX を高速に保つための**設計論**。並列性を「エージェントを増やすこと」ではなく**希少リソースへの競合を減らす設計問題**として扱う、という考え方とロールアウト手順。

## 位置づけ

[[concepts/multi-agent]] の上に「どのエージェントがどの作業を所有し、何をチャットに残し、何をバックグラウンドに回すか」というポリシーを足したもの。OpenClaw が既に持つ [[concepts/session]] ごとの直列化と [[concepts/queue]] のグローバル並列制限を前提に、その上へレーン契約を重ねる。

## 仕組み・ふるまい

### 基本原則：ボトルネックへの競合を減らすときだけ速くなる

希少リソースは **セッションロック**（同一セッションの変更は一度に 1 つ）・**グローバルなモデル容量**（プロバイダー制限を全実行で共有）・**ツール容量**（シェル/ブラウザ/ネットワーク/リポジトリはモデルターンより遅いことがある）・**コンテキスト予算**（長いトランスクリプトが以後を遅くする）・**所有権の曖昧さ**（重複エージェントが容量を浪費）。

### 推奨ロールアウト（3 フェーズ）

1. **レーン契約＋重い作業のバックグラウンド化**（最も安価で詰まりの大半を解消）：各レーンにワークスペース/システムプロンプト内で書面契約を与える――**Owns（所有作業）/Does not own（引き継ぐ作業）/Chat budget（速い回答は即答、長い作業は短く確認してバックグラウンド）/Handoff（移動先・目的・文脈・次アクション）/Tool posture（最小のツールサーフェス）**。
2. **優先度と同時実行制御**：レーンの価値に合わせ `agents.defaults.maxConcurrent`・`subagents.maxConcurrent`/`delegationMode`・`messages.queue`（`mode`/`debounceMs`/`cap`/`drop`）を調整。高優先度は直接/本番エージェント、混雑時はリサーチ/ドラフト/バッチコーディングをバックグラウンドへ。
3. **コーディネーター/トラフィックコントローラー**：複数レーンが動き出したら、所有者追跡・重複検出・引き継ぎルーティング・ブロッカー/結果の表面化を担う小さな調整役を追加。**ここから始めない**（契約のないコーディネーターは混乱を調整するだけ）。

## 設定・使い方の要点

- フェーズ2 の例：`maxConcurrent: 4`、`subagents: { maxConcurrent: 8, delegationMode: "prefer" }`、`messages.queue: { mode: "collect", debounceMs: 1000, cap: 20, drop: "summarize" }`。
- 最小レーン契約テンプレ（`Owns`/`Does not own`/`Chat budget`/`Handoff`/`Tool posture`）をワークスペースに置く。

## 注意点・落とし穴

- 並列化は**実ボトルネックへの競合を減らす場合のみ**スループットを上げる（単にエージェントを増やすと、モデル容量やツール容量で詰まる）。
- レーン契約なしにコーディネーターから始めない。

## 用語と略称

- **レーン（lane）** = 特定の作業領域を所有するエージェント/実行系の筋道
- **レーン契約** = 各レーンの所有/非所有・チャット予算・引き継ぎ・ツール姿勢を書いた合意
- **コーディネーター** = レーン間を調整するトラフィックコントローラー役
- **debounce** = 短時間の入力をまとめて 1 回にする処理

## 関連ページ

- [[concepts/parallel-specialist-lanes]] — 対応する概念ページ
- [[concepts/multi-agent]] / [[concepts/queue]] / [[concepts/session-tool]]
