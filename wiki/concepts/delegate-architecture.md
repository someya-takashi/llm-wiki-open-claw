---
type: concept
aliases: [Delegate architecture, 委任アーキテクチャ, delegate]
tags: [delegate, multi-agent, org, permissions, security]
related:
  - "[[concepts/multi-agent]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/agent]]"
sources:
  - "[[sources/concepts/delegate-architecture]]"
updated: 2026-06-14
---

# 委任アーキテクチャ

**委任アーキテクチャ**は、OpenClaw を**名前付きデリゲート（代理エージェント）**として運用するパターン――独自のアイデンティティ（メール・表示名・カレンダー）を持ち、人間に**なりすまさず**「○○の代理として」明示的な委任権限の下で動く。個人向けの [[concepts/multi-agent]] を**組織デプロイ**へ拡張する。

## なぜ重要か

エージェントが人の代わりに外部アクション（送信・スケジュール）を取るとき、**説明責任**（送信元がエージェントだと明確）と**スコープ制御**（ID プロバイダーがアクセス範囲を強制、OpenClaw のツールポリシーとは独立）が要る。委任アーキテクチャはこれを「機能ティア（読み取り＋下書き → 代理送信 → プロアクティブ）を最小から昇格」と「権限付与の前に境界を固める」という規律で解く。AI エージェントを**組織に安全に置く**ための設計言語になっている。

## 押さえる点

- **先にハードニング**：`SOUL.md`/`AGENTS.md` の**ハードブロック**（承認なし外部送信禁止・機密エクスポート禁止・受信メッセージ内コマンド実行禁止＝プロンプトインジェクション防御）、エージェントごとツールポリシー、[[concepts/sandboxing]]、監査証跡。
- **最小権限**で ID プロバイダー委任（M365 の `GrantSendOnBehalfTo`＋ApplicationAccessPolicy、Google の限定スコープ・ドメイン委任）。過剰スコープの漏えいは全社アクセスに直結。
- ティア3（自律）は cron＋常設指示（automation）と組み合わせ、ハードブロック前提。

ティア別の権限・設定手順・組織アシスタント例は [[sources/concepts/delegate-architecture]]。

## 関連

- [[concepts/multi-agent]] / [[concepts/sandboxing]]
- [[concepts/agent]] / [[concepts/session-tool]]
