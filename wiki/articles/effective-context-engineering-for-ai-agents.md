---
type: article
source_kind: blog
source_url: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
source_path: raw/articles/Effective context engineering for AI agents.md
lang: en
author: [Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, Jeremy Hadfield]
published: 2025
summarized: 2026-06-15
title: "AI エージェントのための効果的なコンテキストエンジニアリング"
tags: [context-engineering, context-rot, compaction, note-taking, sub-agents, just-in-time]
related:
  - "[[concepts/context]]"
  - "[[concepts/compaction]]"
  - "[[concepts/memory]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/system-prompt]]"
translation: "[[articles/translations/effective-context-engineering-for-ai-agents]]"
---

# AI エージェントのための効果的なコンテキストエンジニアリング

> 二次資料（外部）。**Anthropic**（Claude の開発元＝OpenClaw の主要 [[providers/anthropic]]）公式エンジニアリングブログ。英語原文。
> 原典: `raw/articles/Effective context engineering for AI agents.md` ・ https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
> 📖 全文翻訳（一文ずつの忠実訳）：[[articles/translations/effective-context-engineering-for-ai-agents]]

## 一言まとめ

「コンテキストエンジニアリング（context engineering, 推論時にモデルへ渡すトークン集合を精選・維持する営み）」の**総論**。コンテキストを**有限の「注意の予算（attention budget）」**として扱い、**望ましい結果を最大化する最小の高シグナルなトークン集合**を探すという指導原則を軸に、システムプロンプト・ツール・例の設計から、ジャストインタイム検索、長時間軸タスク向けの 3 技法（コンパクション・構造化ノート取り・サブエージェント）までを俯瞰する。

## 位置づけ（OpenClaw との関係）

これは **Anthropic（Claude の開発元）**の総論的な権威記事で、これまで ingest した 3 本――[[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]（Manus の実践則）・[[articles/context-engineering-session-memory]]（短期メモリ）・[[articles/context-engineering-personalization]]（長期メモリ）――の**概念的な土台（anchor）**にあたる。OpenClaw は Claude をモデルプロバイダー（[[providers/anthropic]]）として使うため、この記事の設計思想は OpenClaw の context/memory 系（[[concepts/context]]・[[concepts/compaction]]・[[concepts/session-pruning]]・[[concepts/memory]]・[[concepts/multi-agent]]・[[concepts/system-prompt]]）の**設計の出自そのもの**に近い。OpenClaw 公式仕様ではないが、概念の「なぜ」を最も上流から照らす。

> 注：例示は Anthropic 製品（Claude Code 等）固有。OpenClaw の挙動は各 concept ページ／公式 docs を正とする。

## 中核思想：コンテキストは有限の「注意の予算」

記事の根幹は **context rot（コンテキストの腐敗）**――トークンが増えるほどモデルの正確な想起能力が落ちる現象。Transformer は全トークンが相互に注意を向ける（n トークンで n² のペア関係）ため、長文ほど注意が薄まる。ゆえにコンテキストは「限界収益が逓減する有限資源」であり、**最小の高シグナル集合**を選ぶのが良いコンテキストエンジニアリング。

→ OpenClaw の [[concepts/context]] が「コスト・速度・賢さはウィンドウに何を載せるかで決まる」「ツールは 2 重コスト」と説くのと同じ前提。この記事はその前提の理論的背景（注意の予算・context rot）を与える。

## コンテキストの構成要素ごとの指針

| 構成要素 | 記事の指針 | OpenClaw の対応 |
|---|---|---|
| システムプロンプト | **適切な高度（right altitude）**＝ハードコードの脆さと曖昧さの中間。XML/Markdown でセクション分け、最小だが十分に。 | [[concepts/system-prompt]]（自前組み立て・セクション構成・プロンプトキャッシュ境界） |
| ツール | **トークン効率良く・最小の実用集合**。「どのツールを使うべきか人間が断定できないなら AI も無理」。 | [[concepts/agent]] ツールポリシー／[[concepts/context]] のツール 2 重コスト |
| 例（few-shot） | エッジケースの羅列ではなく**多様で正準的な例**を精選。 | [[concepts/system-prompt]]／[[concepts/context]] |

## ジャストインタイム検索とエージェント的探索

記事は「全データを事前ロード」ではなく、**軽量な識別子（ファイルパス・クエリ・リンク）を保持し、実行時にツールで動的にロードする**「just-in-time」戦略を推す。Claude Code は `head`/`tail`/`glob`/`grep` で大規模データを全ロードせず分析し、`CLAUDE.md` だけ前もって投入する**ハイブリッド**。ファイル名・階層・タイムスタンプ自体がシグナル（漸進的開示）。

→ OpenClaw の [[concepts/memory]]（人間可読な Markdown ファイル＝外部メモリ、オンデマンド取得）・[[concepts/memory-search]]（ベクトル＋BM25）・[[concepts/agent-workspace]]（git 管理）・[[concepts/exec]]（シェルで head/tail/grep）が、この「ファイルシステムを外部記憶として使う」思想を体現する。記事の「CLAUDE.md を前もって投入」は、OpenClaw が [[concepts/system-prompt]] でブートストラップファイル（`AGENTS.md` 等＋`MEMORY.md`）を毎ターン注入する設計と直に対応する。

## 長時間軸タスクの 3 技法 ↔ OpenClaw

記事はコンテキスト汚染に直接対処する 3 技法を挙げ、いずれも OpenClaw に対応物がある。

### ① コンパクション（Compaction）
上限接近時に会話を要約し、新ウィンドウを要約で再初期化。Claude Code は「アーキテクチャ決定・未解決バグ・実装詳細を残し、冗長なツール出力を捨て、直近 5 ファイルを足す」。再現率を最大化→精度を上げる順でプロンプトを調整。**ツール結果クリア**が最も軽く安全な形。

→ [[concepts/compaction]] と**同名・同義**（要約を残し直近は保持）。記事の「ツール結果クリア」は OpenClaw の [[concepts/session-pruning]]（古いツール結果のみメモリ内トリム）に正確に対応する。

### ② 構造化ノート取り（Structured note-taking）
コンテキスト外のメモリにノートを書き、後で引き戻す（NOTES.md・ToDo リスト）。Claude のポケモンプレイが数千ステップの集計・地図・戦略を維持する例。Sonnet 4.5 で**ファイルベースの memory tool** を公開。

→ [[concepts/memory]]（`MEMORY.md`／日次ノート）・[[concepts/progress-drafts]]（進捗の下書き）・[[concepts/active-memory]]（応答前の引き戻し）に対応。「コンテキスト外に書いて後で読む」という OpenClaw メモリの核と一致。

### ③ サブエージェントアーキテクチャ（Sub-agent architectures）
専門サブエージェントがクリーンな文脈で深掘りし、**凝縮した要約（1〜2K トークン）だけ**を親へ返す。詳細な検索文脈はサブ側に隔離、親は統合に集中。

→ [[concepts/multi-agent]]（複数エージェント分離）・[[concepts/delegate-architecture]]（組織デプロイ）・[[concepts/parallel-specialist-lanes]]（並列ワーク）に対応。「関心の分離＋蒸留要約の返却」という設計が共通。

## 全体対応表

**表1**: 記事の論点と OpenClaw の対応機構。

| 記事の論点 | OpenClaw の対応 |
|---|---|
| context rot／注意の予算 | [[concepts/context]] |
| right altitude なシステムプロンプト | [[concepts/system-prompt]] |
| トークン効率の良い最小ツール集合 | [[concepts/agent]]／[[concepts/context]] |
| just-in-time 検索（ファイル参照） | [[concepts/memory]]／[[concepts/memory-search]]／[[concepts/agent-workspace]]／[[concepts/exec]] |
| コンパクション／ツール結果クリア | [[concepts/compaction]]／[[concepts/session-pruning]] |
| 構造化ノート取り／memory tool | [[concepts/memory]]／[[concepts/progress-drafts]]／[[concepts/active-memory]] |
| サブエージェント | [[concepts/multi-agent]]／[[concepts/delegate-architecture]]／[[concepts/parallel-specialist-lanes]] |

## 注意点

- **OpenClaw 公式仕様ではない**が、Claude 開発元＝主要プロバイダー（[[providers/anthropic]]）の総論であり、OpenClaw の context/memory 設計の上流思想として価値が高い。
- 「より賢いモデルほど規範的エンジニアリングは減る」「最もシンプルで効くことをせよ」という見立ては、OpenClaw でも過剰な作り込みを避ける指針として読める。
- コードブロックは無く、図 2 点（プロンプト vs コンテキスト、システムプロンプト較正）は全文翻訳側に参照リンクのみ保持。

## 用語と略称

- **コンテキストエンジニアリング**（context engineering）：推論時にモデルへ渡すトークン集合を精選・維持する営み。プロンプトエンジニアリングの自然な進化形。
- **context rot（コンテキストの腐敗）**：トークン数増加に伴い想起精度が落ちる現象。
- **注意の予算**（attention budget）：モデルが大量コンテキストを解析する際に消費する有限の注意資源。
- **just-in-time（ジャストインタイム）検索**：事前ロードせず実行時に軽量参照からデータを引く戦略。
- **漸進的開示**（progressive disclosure）：探索を通じて関連コンテキストを段階的に発見すること。
- **LLM**（Large Language Model）／ **MCP**（Model Context Protocol, エージェントに外部ツール・データを接続する規格）。

## 関連ページ

- [[concepts/context]] / [[concepts/system-prompt]] / [[concepts/compaction]] / [[concepts/session-pruning]]
- [[concepts/memory]] / [[concepts/memory-search]] / [[concepts/agent-workspace]] / [[concepts/active-memory]] / [[concepts/progress-drafts]] / [[concepts/exec]]
- [[concepts/multi-agent]] / [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]] / [[concepts/agent]] / [[providers/anthropic]]
- 📖 全文翻訳：[[articles/translations/effective-context-engineering-for-ai-agents]]
- 🔗 関連二次資料（コンテキストエンジニアリング群）：[[articles/context-engineering-for-ai-agents-lessons-from-building-manus]] / [[articles/context-engineering-session-memory]] / [[articles/context-engineering-personalization]]
