---
type: article
source_kind: blog
source_url: https://developers.openai.com/cookbook/examples/agents_sdk/context_personalization
source_path: raw/articles/Context Engineering for Personalization - State Management with Long-Term Memory Notes.md
lang: en
author: [OpenAI]
published: 2025
summarized: 2026-06-15
title: "パーソナライゼーションのためのコンテキストエンジニアリング：長期メモリノートによる状態管理"
tags: [context-engineering, long-term-memory, consolidation, memory-injection, personalization, guardrails]
related:
  - "[[concepts/memory]]"
  - "[[concepts/dreaming]]"
  - "[[concepts/active-memory]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/security]]"
translation: "[[articles/translations/context-engineering-personalization]]"
---

# パーソナライゼーションのためのコンテキストエンジニアリング：長期メモリノートによる状態管理

> 二次資料（外部・別プロダクト）。OpenAI Cookbook（developers.openai.com）の解説記事。英語原文。
> 原典: `raw/articles/Context Engineering for Personalization - State Management with Long-Term Memory Notes.md` ・ https://developers.openai.com/cookbook/examples/agents_sdk/context_personalization
> 📖 全文翻訳（一文ずつの忠実訳・全コード収録）：[[articles/translations/context-engineering-personalization]]
> 🔗 前編（短期メモリ＝トリミング/要約）：[[articles/context-engineering-session-memory]]

## 一言まとめ

エージェントを「あなた専用」に感じさせる**長期メモリ＝パーソナライゼーション**を、検索ベースではなく**状態ベース（state-based, 構造化プロフィール＋ノートを権威ある現在状態として持つ）**で実装するパターン。**蒸留（distill, 会話から耐久的シグナルを抽出）→ 統合（consolidate, セッションメモリを重複排除・競合解決・忘却しつつグローバルへ昇格）→ 注入（inject, 各実行の冒頭で関連メモリをシステムプロンプトへ）**のループを、トラベルコンシェルジュを例に OpenAI Agents SDK で示す。

## 位置づけ（OpenClaw との関係）

OpenClaw 公式ドキュメントではなく、**別エコシステム（OpenAI Agents SDK）由来の外部二次資料**。前編 [[articles/context-engineering-session-memory]]（短期メモリ＝1 会話内のトリミング/要約）の**続編**にあたり、本記事は**会話をまたぐ長期メモリ**を扱う。ここで述べられる「蒸留→統合→注入」のループと「忘却は不可欠」という思想は、OpenClaw の [[concepts/memory]]（Markdown による外部メモリ）・[[concepts/dreaming]]（短期シグナルを `MEMORY.md` へ昇格する統合システム）・[[concepts/active-memory]]（応答前のメモリ注入）・[[concepts/system-prompt]]（毎ターンの注入）と**驚くほど同型**。OpenClaw の記憶設計の「なぜ」を、具体コードとともに照らす。

> 注：実装は OpenAI SDK 固有（`RunContextWrapper`・`AgentHooks`・`Responses API`）。OpenClaw の挙動は各 concept ページ／公式 docs を正とし、一般原則の対応として読む。

## メモリライフサイクルと OpenClaw 機構の対応

記事は「2 種類のメモリ × 3 段階のライフサイクル」で整理する。OpenClaw の対応がほぼ 1:1 で付く。

### スコープ：グローバル（長期） vs セッション（短期ステージング）

記事は**ユーザーレベル（global notes, 旅行をまたぐ耐久的な好み）**と**セッションレベル（session notes, 今だけ有効・昇格候補のステージング）**を分け、「デフォルトで将来に効くならグローバル、今だけならセッション」を経験則とする。

→ OpenClaw でも [[concepts/memory]] が `MEMORY.md`（精選された長期）と `memory/YYYY-MM-DD.md`（日次の作業ノート）を分ける構造で対応。さらに「今だけのフォローアップ」は [[concepts/commitments]]（会話に紐づく短期メモリ＝メモリと自動化の中間）が担う。

### ① 蒸留（distillation）↔ メモリ捕捉

記事は `save_memory_note` ツール（**memory-as-a-tool**）でライブに候補メモリを捕捉する。耐久的・実行可能・明示的なものだけを保存し、推測や PII（Personally Identifiable Information, 個人を特定できる情報）は弾く。代替として「セッション後に実行トレースから抽出」も挙げる。

→ OpenClaw では [[concepts/memory]] への書き込みや、[[concepts/commitments]] の「返信後の隠れたバックグラウンド LLM パスで高信頼候補のみ保存」が対応。記事の「ツールスキーマと指示で何を保存するか厳密にスコープする」設計は、ノイズの多い記憶を避ける一般原則。

### ② 統合（consolidation）↔ Dreaming

記事の最重要かつ最もエラーが起きやすい段階。セッション終了時に**非同期**で走り、**重複排除・競合解決（最新が勝つ）・忘却（古い/低信頼のプルーニング）**を経て、耐久的なものだけをグローバルへ昇格する。「**忘却はバグではなく不可欠**」が核心。

→ これは OpenClaw の [[concepts/dreaming]] と**ほぼ同義**：強い短期シグナルを `MEMORY.md` へ、説明可能・レビュー可能に昇格するバックグラウンド統合システム（Cron で夜間スイープ、重み付きスコアとしきい値ゲート）。記事の「過度な統合はポイズニング・記憶損失・幻覚を招く」という警告は、Dreaming が**Deep フェーズだけが昇格を書き、Dream Diary はレビュー用**と分けている理由を裏づける。「統合専用に評価（evals）を作り積極性をチューニング」という助言も、Dreaming の `promote-explain`/`rem-harness` プレビューに通じる。

### ③ 注入（injection）↔ system-prompt / Active Memory

記事は精選メモリを各実行の冒頭で**システムプロンプトへ注入**する（構造化は YAML フロントマター、非構造化は Markdown リスト）。トリミング後にセッションメモリを再注入するフックも持つ。「システムプロンプト内の高シグナルなメモリはレイテンシに極めて効く」。

→ OpenClaw では [[concepts/system-prompt]] が毎ターン `MEMORY.md` を注入し（プロンプトキャッシュ境界の上に安定コンテンツ＝同じ最適化）、[[concepts/active-memory]] が応答前に関連メモリを先回り注入する（レイテンシ直結のトレードオフまで一致）。記事の「メモリは助言的であり現在の指示を上書きしない／明示区切りで包む」は、Active Memory が注入を**「信頼できないコンテキスト（指示として扱うな）」**として扱う設計と同じ発想。

## 優先順位ルールとガードレール ↔ OpenClaw のセキュリティ

記事の優先順位は **最新のユーザー入力 > セッション上書き > グローバル既定**。そしてメモリは「システムプロンプトへ直接入る**価値の高い攻撃面**」として、3 段階すべてにガードレールを要求する：

- **蒸留時**：PII・指示形ペイロードを拒否、スキーマで許可フィールドを制約。
- **統合時**：「捏造なし」厳守、競合は最新が勝つ、重複排除、TTL で減衰。
- **注入時**：`<memories>…</memories>` で包む、優先順位強制、助言扱い。

→ OpenClaw 文脈では [[concepts/security]]（「知能より前にアクセス制御」）・[[concepts/threat-model]]（プロンプトインジェクション／外部コンテンツの untrusted ラップ）・[[concepts/secrets]]（PII/秘密値の外部化）と直結。「コンテキストポイズニング」「指示インジェクション」という攻撃名は OpenClaw の脅威モデルと共通言語。

## ライフサイクル対応表

**表1**: 記事のメモリライフサイクル（左）と OpenClaw の対応機構（右）。

| 記事の段階 | 要点 | OpenClaw の対応 |
|---|---|---|
| スコープ分離 | global（耐久） vs session（今だけ） | [[concepts/memory]]（`MEMORY.md` vs 日次ノート）／[[concepts/commitments]] |
| ① 蒸留 | ツールで高シグナルのみ捕捉 | [[concepts/memory]] 書き込み／[[concepts/commitments]] の隠れ抽出 |
| ② 統合 | 重複排除・競合解決・忘却→昇格 | [[concepts/dreaming]]（Deep 昇格・レビュー可能） |
| ③ 注入 | system prompt へ関連メモリ | [[concepts/system-prompt]]／[[concepts/active-memory]] |
| ガードレール | PII/ポイズニング/指示注入を 3 段で防ぐ | [[concepts/security]]／[[concepts/threat-model]]／[[concepts/secrets]] |
| 評価 | 蒸留/注入/統合をパイプラインで評価 | [[concepts/qa-automation]]／[[concepts/observability]] |

## 注意点

- **OpenClaw 公式仕様ではない**。実装は OpenAI Agents SDK 固有で、対応は一般原則の照合。
- 記事は「状態ベース vs 検索ベース」で**状態ベースを推す**が、OpenClaw のメモリは Markdown ファイル＋[[concepts/memory-search]]（ベクトル＋BM25 のハイブリッド検索）を併用する設計で、両者を排他とはしない点に注意。
- 本記事は前編 [[articles/context-engineering-session-memory]] の `TrimmingSession` を再利用する続編。短期（1 会話内）と長期（会話間）を合わせて読むと全体像が掴める。
- 画像はなく、コードは全文翻訳側に忠実収録。

## 用語と略称

- **状態ベースメモリ**（state-based memory）：構造化フィールド＋ノートを「現在の権威ある状態」として持ち、決定論的に使う方式。検索ベース（過去ログを文書として都度検索）と対比される。
- **蒸留/統合/注入**（distillation / consolidation / injection）：捕捉 → 整理昇格 → コンテキスト投入、というメモリの 3 段階。
- **忘却**（forgetting）：古い・低信頼・重複・置換済みのメモリを意図的にプルーニングすること。
- **コンテキストポイズニング**（context poisoning）／**指示インジェクション**（instruction injection）：メモリ経由でモデルを汚染・誤誘導する攻撃。
- **PII**（Personally Identifiable Information, 個人特定情報）／ **TTL**（Time To Live, 有効期限）／ **ZDR**（Zero Data Retention, データ無保持）／ **SDK**（Software Development Kit）／ **CRM**（Customer Relationship Management）。

## 関連ページ

- [[concepts/memory]] / [[concepts/dreaming]] / [[concepts/commitments]] / [[concepts/active-memory]] / [[concepts/memory-search]]
- [[concepts/system-prompt]] / [[concepts/context]] / [[concepts/session-pruning]]
- [[concepts/security]] / [[concepts/threat-model]] / [[concepts/secrets]] / [[concepts/qa-automation]] / [[concepts/observability]]
- 📖 全文翻訳：[[articles/translations/context-engineering-personalization]]
- 🔗 前編（短期メモリ）：[[articles/context-engineering-session-memory]]　｜　🔗 関連二次資料：[[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]
