---
type: article
source_kind: blog
source_url: https://developers.openai.com/cookbook/examples/agents_sdk/session_memory
source_path: raw/articles/Context Engineering - Short-Term Memory Management with Sessions.md
lang: en
author: [OpenAI]
published: 2025
summarized: 2026-06-15
title: "コンテキストエンジニアリング：Session による短期メモリ管理"
tags: [context-engineering, session, trimming, summarization, compaction, evals]
related:
  - "[[concepts/session]]"
  - "[[concepts/session-pruning]]"
  - "[[concepts/compaction]]"
  - "[[concepts/context]]"
  - "[[concepts/memory]]"
translation: "[[articles/translations/context-engineering-session-memory]]"
---

# コンテキストエンジニアリング：Session による短期メモリ管理

> 二次資料（外部・別プロダクト）。OpenAI Cookbook（developers.openai.com）の解説記事。英語原文。
> 原典: `raw/articles/Context Engineering - Short-Term Memory Management with Sessions.md` ・ https://developers.openai.com/cookbook/examples/agents_sdk/session_memory
> 📖 全文翻訳（一文ずつの忠実訳・全コード収録）：[[articles/translations/context-engineering-session-memory]]

## 一言まとめ

長時間・複数ターンのエージェント会話で**コンテキスト（context, モデルが一度に注意を向けられる入力＋出力トークンの総ウィンドウ）**を破綻させないための短期メモリ管理を、OpenAI Agents SDK の `Session` オブジェクトを題材に、**トリミング（trimming, 古いターンを丸ごと捨て直近 N ターンだけ残す）**と**要約（summarization, 古い履歴を合成メッセージへ圧縮する）**の 2 技法で対比して解説する。

## 位置づけ（OpenClaw との関係）

OpenClaw 公式ドキュメントではなく、**別エコシステム（OpenAI Agents SDK）由来の外部二次資料**。ただし扱う問題――「履歴が伸びてもコンテキストウィンドウを破綻させず、コスト・速度・一貫性を保つ」――は、OpenClaw の [[concepts/session]]（会話の器）・[[concepts/session-pruning]]（刈り込み）・[[concepts/compaction]]（要約圧縮）・[[concepts/context]]（窓に載せる全体）が解いている問題と**そのまま同型**。本記事は、これら OpenClaw 機構の設計判断の「なぜ」と一般的トレードオフを、具体コードとともに照らす。[[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]（Manus 流コンテキストエンジニアリング）と対をなす二次資料。

> 注：本記事は実装が OpenAI SDK 固有（`Session`/`Responses API`/`previous_response_id` 等）。OpenClaw の挙動は各 concept ページ／公式 docs を正とし、ここでは「一般原則の対応」として読む。

## 2 技法と OpenClaw 機構の対応

### トリミング（直近 N ターンを残す） ↔ OpenClaw のプルーニング

記事の `TrimmingSession` は、書き込み/読み取りのたびに履歴を後方走査し、**直近 N 個の user メッセージ**とその後続（assistant 返信・ツール呼び出し/結果）だけを「ターン境界を壊さずに」残し、それより前を丸ごと捨てる。利点は**決定論的・追加レイテンシゼロ・直近の忠実性**、欠点は**長距離の記憶を唐突に失う（健忘）**こと。

→ OpenClaw で近いのは [[concepts/session-pruning]]。ただし**粒度が違う**：OpenClaw のプルーニングは「**古いツール結果だけ**をメモリ内でトリム（会話本文は保持・ディスクのトランスクリプトは不変）」するのに対し、記事のトリミングは「**ターンを丸ごと**捨てる」。どちらも「直近を残し古いものを軽くする」発想だが、OpenClaw は会話文を捨てない分だけ健忘リスクを抑える設計といえる。プルーニングが特に効くのは Anthropic のプロンプトキャッシュ（書き込みサイズ削減＝コスト削減）で、記事の「より低いレイテンシ＆コスト」と動機が一致する。

### 要約（古い履歴を合成ペアに圧縮） ↔ OpenClaw の Compaction

記事の `SummarizingSession` は、実 user ターンが `context_limit` を超えると、**直近 `keep_last_n_turns` ターンをそのまま残し、それ以前を「合成 user→assistant ペア」へ要約**する。具体的には `user: "Summarize the conversation we had so far."`（自然な流れを保つシャドウプロンプト）＋ `assistant: {生成要約}` を保持領域の先頭へ注入し、`metadata.synthetic == True` で印を付ける。要約プロンプトは「次の担当者へ案件を引き継ぐつもりで」設計し、矛盾チェック・時系列・幻覚抑制（不明は `UNVERIFIED`）・構造化見出しを指示する。

→ これは OpenClaw の [[concepts/compaction]] と**ほぼ同じ設計**：古い会話を要約にまとめ、直近メッセージはそのまま残し、要約はトランスクリプトに保存、完全履歴はディスクに残す。OpenClaw が Compaction 前に [[concepts/memory]] へ重要メモを退避する「メモリフラッシュ」を挟むのは、記事がいう**要約損失・コンテキストポイズニング（context poisoning, 誤った事実が要約に入り将来へ伝播する汚染）**を抑える工夫に対応する。「要約専用モデルを選ぶ」点（記事の Model Choice）も OpenClaw の `compaction.model` と一致。

### Session オブジェクト ＝ メモリの器 ↔ OpenClaw の Session

記事は「`session.run("...")` を繰り返すだけで SDK がコンテキスト長・履歴・継続性を握る」とし、**セッションをメモリオブジェクト**と位置づける。OpenClaw でも [[concepts/session]] が会話文脈の器であり、状態は [[components/gateway]] が所有し、[[concepts/agent-loop]] はセッションキー単位で直列化される。「会話の連続性をどこが持つか」という設計思想が共通する。

## トリミング vs 要約：トレードオフ表

**表1**: 記事の比較（左 2 列）に OpenClaw の対応機構（右列）を併記。

| 観点 | トリミング（last-N） | 要約（古い→生成要約） | OpenClaw の対応 |
|---|---|---|---|
| レイテンシ/コスト | 最低（追加呼び出しなし） | 要約時に増 | [[concepts/session-pruning]]（軽量・キャッシュ節約） / [[concepts/compaction]]（要約コスト有） |
| 長距離の想起 | 弱い（ハード打ち切り） | 強い（コンパクトに持ち越し） | プルーニングは会話文を残すため健忘を緩和 |
| リスク | コンテキスト喪失 | 歪み/ポイズニング | Compaction 前のメモリフラッシュで緩和 |
| 最適 | ツール中心・短い流れ | 長いスレッド・状態保持 | 両者を相互補完で併用 |

## 評価（Evals）の指針

記事は締めで「コンテキストエンジニアリングでも**評価こそすべて**」とし、軽量ハーネス案を挙げる：ベースライン差分、LLM-as-Judge（要約品質採点）、トランスクリプト再生（ID/エンティティ完全一致・次ターン精度）、エラー回帰追跡、トークン圧迫チェック。

→ OpenClaw 文脈では [[concepts/qa-automation]]（品質自動化）や [[concepts/observability]]（可観測性）と接続する発想。「メモリ管理が壊れていないかを測る」観点は、Compaction/プルーニングを運用に乗せる際のチェックリストになる。

## 注意点

- **OpenClaw 公式仕様ではない**。実装は OpenAI Agents SDK 固有で、対応はあくまで一般原則の照合。
- 記事の「トリミング」は**ターン単位の破棄**で、OpenClaw のプルーニング（ツール結果のみ）とは粒度が異なる点に注意。
- 画像（cookbook の図 2 点）は本要約に取り込んでいない。全文翻訳側にも参照リンクのみ保持。

## 用語と略称

- **コンテキストポイズニング**（context poisoning）：誤った事実が要約や履歴に混入し、以後のターンへ伝播して挙動を汚染すること。
- **トリミング**（trimming）：古いターンを丸ごと捨て直近 N ターンだけ残す手法。
- **要約/圧縮**（summarization / compression）：古い履歴を短い合成メッセージへまとめる手法。
- **シャドウプロンプト**（shadow prompt）：会話の自然な流れを保つために挿入する、要約要求の合成 user 発話。
- **SDK**（Software Development Kit）／ **API**（Application Programming Interface）／ **RAG**（Retrieval-Augmented Generation, 検索拡張生成）／ **LLM-as-Judge**（モデルを採点者に使う評価法）。

## 関連ページ

- [[concepts/session]] / [[concepts/session-pruning]] / [[concepts/compaction]] / [[concepts/context]]
- [[concepts/memory]] / [[concepts/context-engine]] / [[concepts/agent-loop]] / [[concepts/qa-automation]] / [[concepts/observability]] / [[concepts/model-providers]]
- 📖 全文翻訳：[[articles/translations/context-engineering-session-memory]]
- 🔗 対をなす二次資料：[[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]
