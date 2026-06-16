---
type: article
source_kind: blog
source_url: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
source_path: raw/articles/Effective harnesses for long-running agents.md
lang: en
author: [Justin Young]
published: 2025
summarized: 2026-06-15
title: "長時間実行エージェントのための効果的なハーネス"
tags: [long-running-agents, harness, multi-context-window, progress-file, git, testing]
related:
  - "[[concepts/agent-workspace]]"
  - "[[concepts/memory]]"
  - "[[concepts/compaction]]"
  - "[[concepts/session]]"
  - "[[concepts/multi-agent]]"
translation: "[[articles/translations/effective-harnesses-for-long-running-agents]]"
---

# 長時間実行エージェントのための効果的なハーネス

> 二次資料（外部）。**Anthropic**（Claude の開発元＝OpenClaw の主要 [[providers/anthropic]]）公式エンジニアリングブログ。英語原文。
> 原典: `raw/articles/Effective harnesses for long-running agents.md` ・ https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents
> 📖 全文翻訳（一文ずつの忠実訳）：[[articles/translations/effective-harnesses-for-long-running-agents]]
> 🔗 姉妹記事（総論）：[[articles/effective-context-engineering-for-ai-agents]]

## 一言まとめ

数時間〜数日に及ぶタスクを、**複数のコンテキストウィンドウ（＝離散セッション）をまたいで**一貫して進めるための**ハーネス（harness, エージェントを駆動する足場一式）**設計。鍵は「**前のセッションの記憶を持たない新セッションへ、作業状態を橋渡しする成果物（artifacts）を残す**」こと。Claude Agent SDK での実験から、**初期化エージェント（環境を整える初回専用プロンプト）＋コーディングエージェント（毎回 1 機能だけ進め、git コミットと進捗ファイルを残す）**の二段構えを提案する。

## 位置づけ（OpenClaw との関係）

**Anthropic（Claude 開発元＝OpenClaw の主要プロバイダー [[providers/anthropic]]）**の実践記事で、姉妹記事 [[articles/effective-context-engineering-for-ai-agents]] が説いた「長時間軸タスクの 3 技法（compaction・ノート取り・サブエージェント）」を、**具体的なハーネスとして実装**したもの。中心メッセージ「**コンパクションだけでは不十分。コンテキスト外に橋渡し成果物（進捗ファイル＋git 履歴）を残せ**」は、OpenClaw の [[concepts/agent-workspace]]（git 管理されるワークスペース）・[[concepts/memory]]（Markdown による外部メモリ）・[[concepts/session]]（離散セッション）・[[concepts/compaction]] の設計判断を裏から照らす。OpenClaw 公式仕様ではないが、Claude を動かす側の知見として価値が高い。

> 比喩：記事は長時間タスクを「**シフト制の交代勤務**」に喩える。各シフトの作業者（セッション）は前任の記憶を持たないので、引き継ぎノート（進捗ファイル）と作業履歴（git）を残すことで連続性を作る。

## 中核：コンパクションだけでは足りない

記事は、Claude Agent SDK が compaction（要約圧縮）を持っていても、高レベルなプロンプトだけでは本番品質に届かない、と観察する。失敗は 2 パターン：
1. **一度にやりすぎ（one-shot）**：実装途中でコンテキスト切れ→次セッションが半端な状態から推測で再開。
2. **早すぎる完了宣言**：進捗を見て「もう終わった」と誤判断。

→ これは OpenClaw の [[concepts/compaction]] が「要約は次エージェントへ常に完璧な指示を渡すわけではない」限界を持つことと整合し、補完として [[concepts/memory]]（明示的に残す外部ノート）や [[concepts/agent-workspace]]（git 履歴）が要る、という設計を裏づける。

## 解決策＝橋渡し成果物を残すハーネス

### 初期化エージェント（initializer）↔ ブートストラップ
初回専用プロンプトで環境を整える：**機能リスト（feature list, 200 超の E2E 機能を `passes:false` で列挙した JSON）**、進捗ログ `claude-progress.txt`、初回 git コミット、`init.sh`（開発サーバー起動）。機能リストに JSON を選ぶのは「Markdown より勝手に書き換えにくい」ため。

→ OpenClaw では [[concepts/agent-workspace]] のブートストラップファイル群（`AGENTS.md`/`TOOLS.md` 等）や [[concepts/system-prompt]] の初回注入が、近い「環境を整える」役割を担う。記事の脚注いわく、初期化とコーディングの 2 つは**同じハーネス・同じツール・同じシステムプロンプトで、初期プロンプトだけが違う**（厳密にはマルチエージェントではない）。

### コーディングエージェント（coding）↔ ワークスペース＋メモリ
毎セッションで **1 機能だけ**進め、終わりに**記述的メッセージで git コミット**し、**進捗ファイルに要約**を書く。次セッションは `pwd`→進捗ファイル/git ログを読む→最優先の未完機能を選ぶ、という順で状況把握。

→ これは OpenClaw の [[concepts/agent-workspace]]（**private git リポジトリでバックアップする運用が推奨**＝記事の git 履歴活用と同じ）と [[concepts/memory]]（`MEMORY.md`／日次ノートで会話をまたいで持続）に**直接対応**。「コンテキスト外に書いて次回読み返す」という構造化ノート取りの実地版。

### テスト（自己検証）↔ ツール使用
Claude は明示指示がないと「動かないのに完了」と印付けしがち。**ブラウザ自動化（Puppeteer MCP）で人間のように E2E 検証**させると激変。ただし vision やブラウザ自動化の限界（ネイティブのアラートモーダルが見えない等）でバグが残る領域もある。

→ OpenClaw 文脈では、エージェントが自分の成果を検証する手段は [[concepts/exec]]（シェルで `curl`/テスト実行）・[[concepts/media-understanding]]（スクリーンショットの視覚理解）・MCP（[[concepts/skills]]/外部ツール接続）に対応する。※OpenClaw の [[concepts/qa-automation]] は「OpenClaw 自身をメンテナーが検証する」別物なので混同しない。

## 失敗モードと解決策（対応表）

**表1**: 記事の 4 失敗モードと OpenClaw 側の対応概念。

| 失敗モード | 記事の解決策 | OpenClaw の対応 |
|---|---|---|
| プロジェクト完了を早まって宣言 | 機能リスト JSON を作り 1 機能ずつ | （タスク分割の発想）[[concepts/memory]] にゴール記録 |
| 環境を未文書化・バグ状態で放置 | git リポ＋進捗ノート、開始時に基本テスト | [[concepts/agent-workspace]]（git）＋[[concepts/memory]] |
| 機能を未テストで完了印付け | 全機能を自己検証してから passing | [[concepts/exec]]／[[concepts/media-understanding]]（E2E 検証） |
| アプリ実行方法の調査に浪費 | `init.sh` を用意し開始時に読む | [[concepts/agent-workspace]] のブートストラップ |

## 今後の課題 ↔ マルチエージェント

記事は「単一汎用エージェント vs マルチエージェント」を未解決とし、**テスト/QA/クリーンアップの専門エージェント**がサブタスクで勝るかもしれないと示唆する。

→ OpenClaw の [[concepts/multi-agent]]（複数エージェント分離）・[[concepts/delegate-architecture]]（組織デプロイ）・[[concepts/parallel-specialist-lanes]]（並列の専門レーン）が、この「専門エージェントへの分業」を実現する枠組みにあたる。

## 注意点

- **OpenClaw 公式仕様ではない**。例示は Claude Agent SDK / Claude Code 固有（`init.sh`・`claude-progress.txt`・Puppeteer MCP）。OpenClaw の挙動は各 concept ページ／公式 docs を正とする。
- 記事は「初期化／コーディング」を別エージェントと呼ぶが**実体は同一ハーネス・初期プロンプト違い**（脚注）。OpenClaw の [[concepts/multi-agent]] とは粒度が異なる点に注意。
- コードブロック 2 点（機能リスト JSON・セッション開始メッセージ例）と図 1 点（GIF）は全文翻訳側に忠実収録／参照のみ。

## 用語と略称

- **ハーネス**（harness）：エージェントを駆動する足場一式（プロンプト・ツール・スクリプト・進捗管理の仕組み）。
- **初期化エージェント／コーディングエージェント**：初回環境構築用と、以降の漸進実装用。実体は同一ハーネスで初期プロンプトのみ異なる。
- **機能リスト**（feature list）：E2E 機能を `passes` フラグ付きで列挙した検証用 JSON。
- **きれいな状態**（clean state）：メインブランチへマージ可能な、バグなし・整然・文書化済みのコード状態。
- **MCP**（Model Context Protocol, エージェントに外部ツール・データを接続する規格。本記事では Puppeteer MCP＝ブラウザ自動化）／ **SDK**（Software Development Kit）／ **E2E**（End-to-End, 端から端までの動作検証）／ **QA**（Quality Assurance）。

## 関連ページ

- [[concepts/agent-workspace]] / [[concepts/memory]] / [[concepts/session]] / [[concepts/compaction]] / [[concepts/system-prompt]]
- [[concepts/exec]] / [[concepts/media-understanding]] / [[concepts/skills]] / [[concepts/agent-loop]]
- [[concepts/multi-agent]] / [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]] / [[providers/anthropic]]
- 📖 全文翻訳：[[articles/translations/effective-harnesses-for-long-running-agents]]
- 🔗 姉妹記事（総論）：[[articles/effective-context-engineering-for-ai-agents]]　｜　関連二次資料：[[articles/context-engineering-personalization]]
