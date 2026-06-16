---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/automation/standing-orders
source_path: raw/docs/automation/standing-orders.md
doc_section: automation
title: "常時指示（Standing Orders）"
ingested: 2026-06-14
tags: [standing-orders, persistent-instructions, programs, automation, escalation]
related:
  - "[[concepts/automation]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/heartbeat]]"
---

# 常時指示（Standing Orders）（解説）

> 原典: `raw/docs/automation/standing-orders.md` ・ https://docs.openclaw.ai/ja-JP/automation/standing-orders

## 一言まとめ

すべてのセッションに**自動注入される常設の指示（プログラム）**。「返信前に常にコンプライアンスを確認」「週次サイクルを回す」のような、エージェントが毎回従うべき手順・規則・エスカレーション方針を宣言する。

## 位置づけ

[[concepts/automation]] の「常に効く」担当。cron（正確なタイミング）とは対照的に、Standing Orders は**いつ実行するか**でなく**毎回どう振る舞うか**を定める。注入先は [[concepts/system-prompt]]、定期実行のトリガーは [[concepts/heartbeat]]/[[concepts/cron]] と組み合わせる。

## 仕組み・ふるまい

- **プログラム構造**：`## Program: <名前>` 単位で、実行ステップ・規則（content rules 等）・「やってはいけないこと」・エスカレーション行列を Markdown で書く。
- **マルチプログラム**：ドメイン別（週次/月次/オンデマンド）に複数プログラムを並べ、共通のエスカレーション規則を持てる。
- 実行・検証・報告パターンで「実行→検証→報告」を毎サイクル回す。

## 設定・使い方の要点

- cron との使い分け：cron は「いつ」を予約、Standing Orders は「何を毎回」。両者を組み合わせて週次レポート等を回す。
- ベストプラクティス：具体的な条件→アクション→エスカレーションを書く。曖昧な指示や過剰な自動化を避ける。

## 注意点・落とし穴

- すべてのセッションに注入されるためトークン予算と相互作用する（[[concepts/context]]）。広すぎる指示は全会話に影響する。

## 用語と略称

- **Standing Orders（常設指示）** = 全セッションに注入される恒常的な指示
- **プログラム（program）** = 1 ドメイン分の手順・規則のまとまり
- **エスカレーション行列** = 条件→アクション→上位通知の対応表

## 関連ページ

- [[concepts/automation]] / [[concepts/system-prompt]] / [[concepts/heartbeat]]
- [[concepts/cron]] / [[concepts/context]] / [[sources/concepts/soul]]
