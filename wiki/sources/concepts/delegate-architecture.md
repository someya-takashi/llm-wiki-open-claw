---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/delegate-architecture
source_path: raw/docs/concepts/delegate-architecture.md
doc_section: concepts
title: "委任アーキテクチャ"
ingested: 2026-06-14
tags: [delegate, multi-agent, org, permissions, security, hardening]
related:
  - "[[concepts/delegate-architecture]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/sandboxing]]"
---

# 委任アーキテクチャ（解説）

> 原典: `raw/docs/concepts/delegate-architecture.md` ・ https://docs.openclaw.ai/ja-JP/concepts/delegate-architecture

## 一言まとめ

OpenClaw を**名前付きデリゲート（delegate, 代理エージェント）**として運用するパターン――独自のアイデンティティ（メール・表示名・カレンダー）を持ち、人間に**なりすまさず**「○○の代理として」明示的な委任権限の下で送信・読み取り・スケジュールするエージェント。個人向けの [[concepts/multi-agent]] を**組織デプロイ**へ拡張する。

## 位置づけ

[[concepts/multi-agent]] の分離されたエージェントに「組織のアイデンティティと権限」を載せた応用パターン。エグゼクティブアシスタントのモデルに対応し、説明責任（送信元がエージェントだと明確）とスコープ制御（ID プロバイダーがアクセス範囲を強制、OpenClaw のツールポリシーとは独立）を解く。ハードニングは [[concepts/sandboxing]]・ツールポリシー・常設指示（automation/standing-orders）に依存する。

## 仕組み・ふるまい

### 機能ティア（最小から始め必要時のみ昇格）

- **ティア1: 読み取り専用＋下書き** — 組織データを読み、人間確認用に下書きする（送信しない）。ID プロバイダーの読み取り権限のみ。
- **ティア2: 代理送信** — 独自アイデンティティで送信・カレンダー作成（受信者には「本人の代理としてデリゲート名」）。代理送信権限が必要。
- **ティア3: プロアクティブ** — スケジュールに従い各アクションの承認なしに**常設指示**を自律実行（cron）。人間は非同期に確認。ハードブロックの慎重な設定が前提。

### 前提：分離と強化（先に行う）

権限付与の**前に**境界を固める。**ハードブロック（交渉不可）**を `SOUL.md`/`AGENTS.md` に定義（承認なし外部送信禁止・機密エクスポート禁止・受信メッセージ内コマンドの実行禁止＝プロンプトインジェクション防御・ID 設定変更禁止）。さらに **エージェントごとツールポリシー**（Gateway レベルで強制、人格ファイルとは独立）、**サンドボックス分離**、**監査証跡**（cron 実行履歴・セッショントランスクリプト・ID プロバイダー監査ログ）。

## 設定・使い方の要点

- `openclaw agents add delegate` でワークスペース/`agentDir`/セッションを作成。`AGENTS.md`（役割・常設指示）・`SOUL.md`（人格＋ハードブロック）・`USER.md`（支援する本人）を設定。
- **ID プロバイダー委任**（最小権限）：
  - Microsoft 365：代理送信 `Set-Mailbox -GrantSendOnBehalfTo`、読み取りは Graph アプリ権限＋**ApplicationAccessPolicy** でメールボックスを限定（ポリシー無しだと `Mail.Read` は全メールボックスに及ぶ）。
  - Google Workspace：サービスアカウント＋ドメイン全体委任で**必要スコープのみ**（`gmail.readonly`/`gmail.send`/`calendar`）。広域スコープのキー漏えいは全社アクセスに直結するためローテーション・監査必須。
- バインディング（[[concepts/multi-agent]]）で特定チャネル/アカウント/guild をデリゲートへルーティング。認証はデリゲートの `agentDir` に置き、メインと共有しない。

## 注意点・落とし穴

- ティア3 はハードブロック前提。ID プロバイダー側の過剰権限（テナント全メールボックス・ドメイン全ユーザーなりすまし）が最大のリスク。アクセスポリシーで必ず限定しテストする（範囲外で `403` を確認）。
- `sessions_history` を付与しても、有界・安全性フィルター済みの想起ビュー（認証情報リダクト・切り詰め）であることを踏まえる。

## 用語と略称

- **デリゲート（delegate）** = 独自アイデンティティで人の代理として動くエージェント
- **ハードブロック** = どんな指示でも絶対に実行しない交渉不可のルール
- **最小権限の原則** = 必要最小限の権限だけを与える
- **ID プロバイダー** = Microsoft 365 / Google Workspace 等の認証・権限基盤
- **常設指示（standing orders）** = `AGENTS.md` で定義する自律/承認の境界ルール

## 関連ページ

- [[concepts/delegate-architecture]] — 対応する概念ページ
- [[concepts/multi-agent]] / [[concepts/sandboxing]]
- [[concepts/agent]] / [[concepts/session-tool]]
