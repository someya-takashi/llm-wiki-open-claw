---
type: article
source_kind: blog
source_url: https://manus.im/ja/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
source_path: raw/articles/AIエージェントのためのコンテキストエンジニアリング：Manus構築から得た教訓.md
lang: ja
author: [Yichao "Peak" Ji]
published: 2025
summarized: 2026-06-15
title: "AIエージェントのためのコンテキストエンジニアリング：Manus構築から得た教訓"
tags: [context-engineering, kv-cache, memory, tool-use, agent-loop, few-shot]
related:
  - "[[concepts/context]]"
  - "[[concepts/context-engine]]"
  - "[[concepts/compaction]]"
  - "[[concepts/memory]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/agent]]"
  - "[[concepts/active-memory]]"
---

# AIエージェントのためのコンテキストエンジニアリング：Manus構築から得た教訓

> 二次資料（外部・別プロダクト）。Manus（現 Meta）チームの公式ブログ記事の日本語版。
> 原典: `raw/articles/AIエージェントのためのコンテキストエンジニアリング：Manus構築から得た教訓.md` ・ https://manus.im/ja/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus

## 一言まとめ

AI エージェント製品 **Manus** を作る中で得た、**コンテキストエンジニアリング（context engineering, モデルに毎ターン何をどう載せるかを設計する営み）**の 6 つの実践的教訓。モデルを再学習せず、**文脈内学習（in-context learning, モデルの重みを変えずプロンプト内の例・指示で振る舞いを変える性質）**の上にエージェントを築く立場から書かれている。

## 位置づけ（OpenClaw との関係）

これは OpenClaw 公式ドキュメントではなく、**別プロダクト（Manus）由来の二次資料**。ただし扱う主題は「AI エージェントのコンテキスト設計の一般原則」であり、OpenClaw が持つ [[concepts/context]]（モデルへ送る全体）・[[concepts/context-engine]]（その組み立てを握る差し替え可能スロット）・[[concepts/compaction]]（圧縮）・[[concepts/memory]]（Markdown ファイルによる外部メモリ）の**設計思想の「なぜ」を裏側から照らす**。OpenClaw の公式仕様としてではなく、**設計の背景理解**として読むとよい。

> 注：原文は英語（著者 Yichao "Peak" Ji、Manus 共同創業者、2025 年公開）。本 wiki が保持するのは公式日本語版のため、全文翻訳ページは作らず本ページの要約のみとする。

## 6 つの教訓と OpenClaw 概念への接続

### 1. KV キャッシュを中心に設計する

エージェントは「入力（増え続ける文脈）に対し短い構造化出力（関数呼び出し）」を繰り返すため、入力:出力のトークン比が極端（Manus で約 100:1）。同一プレフィックスは **KV キャッシュ（KV-cache, Key-Value Cache: 自己回帰生成で前方トークンの内部計算を再利用するキャッシュ）**が効き、**TTFT（Time To First Token, 最初のトークンが返るまでの待ち時間）**と推論コストを大幅に下げる（Claude Sonnet ではキャッシュ済み入力が約 1/10 の単価）。だから①プレフィックスを安定させる（システムプロンプト先頭に秒刻みのタイムスタンプを置かない）②コンテキストは追加専用（append-only）にし JSON のキー順を決定的にする③必要ならキャッシュ区切りを明示、が要点。

→ OpenClaw では、毎ターン注入される [[concepts/system-prompt]] と [[concepts/context]] の組み立て安定性がそのままキャッシュ効率に効く。プロンプトキャッシュの経済性はプロバイダー依存（[[concepts/model-providers]]）。「プレフィックスを壊さない」という原則は、OpenClaw が履歴をトランスクリプトとして**追記的に**保持し、[[concepts/compaction]] で初めて畳む設計と整合する。

### 2. マスクし、削除しない

ツールが増えると（MCP＝Model Context Protocol, エージェントに外部ツール・データを接続する標準規格、の普及で特に）アクション空間が膨張し、モデルはかえって誤選択しやすくなる。途中でツール定義を足し引きすると、定義は文脈前方にあるため**後続全体の KV キャッシュが無効化**し、過去の参照と食い違って幻覚を招く。Manus はツールを**消さず**、デコード時に**ロジットをマスク**（ステートマシンで可用性制御）し、ツール名に一貫接頭辞（`browser_` `shell_`）を付けて状態ごとに選択肢を絞る。

→ OpenClaw の [[concepts/agent]] ツールポリシーも、**定義を安定させたまま `deny` 優先で可否を制御する**思想に近い。ツールは [[concepts/context]] が指摘する「2 重コスト（プロンプト内リスト＋送信スキーマ JSON）」を持つため、安易な動的増減はコストとキャッシュの両面で不利、という本記事の指摘は OpenClaw でも有効。

### 3. ファイルシステムをコンテキストとして使う

128K トークン超の窓でも、巨大な観測（Web ページ・PDF）ではすぐ溢れ、長文ほど性能低下とコスト増を招く。不可逆な圧縮は将来必要になる情報を失うリスクがある。Manus は**ファイルシステムを無制限・永続・エージェントが直接操作できる外部メモリ**として扱い、URL やパスを残せば本文はいつでも復元できる「**可逆な圧縮**」を徹底する。

→ これは OpenClaw の [[concepts/memory]]（`MEMORY.md`・日次ノート等の**人間も読める Markdown ファイル＝外部メモリ**）と [[concepts/agent-workspace]]（git 管理されるワークスペース）の設計そのもの。「隠れた状態を持たず、ディスク上のファイルに外部化する」という透明性の思想が一致する。組み立て戦略の実装点は [[concepts/context-engine]]。

### 4. 暗唱（recitation）で注意を操作する

Manus は複雑タスクで `todo.md` を作り、進行に応じて書き直してチェックを入れる。これは**目標をコンテキスト末尾に「暗唱」**し、グローバルな計画を直近の注意範囲に置くことで「**中間での迷子（lost-in-the-middle）**」と目標ずれを防ぐ意図的メカニズム（平均 50 回前後のツール呼び出しに及ぶ長ループで効く）。

→ OpenClaw では、応答前に関連メモリを先回りで表面化する [[concepts/active-memory]] や、[[concepts/memory]] への要約・[[concepts/progress-drafts]] が近い役割を担う。「重要事項を文脈の効く位置に再提示し続ける」という発想の一般形がこの教訓。

### 5. 間違ったものも残しておく

失敗（幻覚・ツールエラー・エッジケース）はループの一部。トレースを消して再試行すると**証拠が消え、モデルが適応できない**。失敗したアクションと観測・スタックトレースを**コンテキストに残す**と、モデルは暗黙に内部の事前確率を更新し同じ誤りを繰り返しにくくなる。エラー回復こそ真のエージェント性の指標だ、という主張。

→ OpenClaw の [[concepts/agent-loop]]（受信→推論→ツール→観測の反復）と [[concepts/retry]] に通じる。とくに [[concepts/compaction]] が「要約を残しつつ直近は保持」「過度に積極的な圧縮を避ける」設計である点は、本記事の「不可逆圧縮を避け失敗の証拠を消すな」と方向性が一致する。

### 6. Few-shot に騙されない

**Few-shot プロンプティング（few-shot prompting, 少数の入出力例を示して出力を導く手法）**は、エージェントでは裏目に出ることがある。文脈が類似の行動-観測ペアで埋まると、モデルは最適でなくてもそのパターンを**模倣**し、ドリフト・過度の一般化・幻覚を招く（例：20 件の履歴書を惰性で同じ手順で処理）。対策は**多様性**——シリアライズ書式・言い回し・順序に少量のノイズを入れてパターンを崩す。

→ コンテキストが均一なほどエージェントは脆くなる、という指摘は、OpenClaw で履歴やテンプレートを組み立てる [[concepts/context]] / [[concepts/system-prompt]] 設計でも留意すべき一般原則。

## 教訓 → OpenClaw 対応表

**表1**: Manus の各教訓と、OpenClaw で対応する機構（一般原則の対応であり、実装が同一という意味ではない）。

| Manus の教訓 | 核心 | OpenClaw で対応する機構 |
|---|---|---|
| ① KV キャッシュ中心 | プレフィックス安定・append-only | [[concepts/context]] / [[concepts/system-prompt]] / [[concepts/model-providers]] |
| ② マスクし削除しない | ツール定義は固定し可否で制御 | [[concepts/agent]] ツールポリシー（`deny` 優先） |
| ③ FS を外部メモリに | 可逆圧縮・無制限・永続 | [[concepts/memory]] / [[concepts/agent-workspace]] / [[concepts/context-engine]] |
| ④ 暗唱で注意操作 | 目標を効く位置へ再提示 | [[concepts/active-memory]] / [[concepts/progress-drafts]] |
| ⑤ 間違いを残す | 失敗の証拠で適応 | [[concepts/agent-loop]] / [[concepts/retry]] / [[concepts/compaction]] |
| ⑥ Few-shot に注意 | 多様性で模倣ドリフト回避 | [[concepts/context]] / [[concepts/system-prompt]] |

## 注意点

- 本記事は **OpenClaw の公式仕様ではない**。別プロダクト（Manus）の経験則であり、上記対応はあくまで「一般原則の照らし合わせ」。OpenClaw 固有の挙動は各 concept ページ／公式 docs を正とする。
- 一部は自己ホスト推論（**vLLM**, 高速 LLM 推論サーバー）前提の話（プレフィックスキャッシュ有効化、セッション ID でのルーティング）で、推論 API 利用時はプロバイダー任せになる点に注意。
- 図（記事中の cloudfront 画像）は本要約には取り込んでいない。詳細図は原典参照。

## 用語と略称

- **KV-cache**（Key-Value Cache）：自己回帰生成で前方トークンの内部表現を再利用し、再計算を省くキャッシュ。
- **TTFT**（Time To First Token）：最初の出力トークンが返るまでの待ち時間。
- **LLM**（Large Language Model, 大規模言語モデル）。
- **MCP**（Model Context Protocol）：エージェントに外部ツール・データを接続する標準規格。
- **RAG**（Retrieval-Augmented Generation）：検索で取得した文書を文脈に足して生成する手法。
- **SSM**（State Space Model, 状態空間モデル）：Transformer と異なり全注意を持たない系列モデル。
- **vLLM**：高スループットな LLM 推論サーバー実装。
- **PMF**（Product-Market Fit, 製品と市場の適合）。

## 関連ページ

- [[concepts/context]] / [[concepts/context-engine]] / [[concepts/compaction]] / [[concepts/system-prompt]]
- [[concepts/memory]] / [[concepts/active-memory]] / [[concepts/agent-workspace]]
- [[concepts/agent]] / [[concepts/agent-loop]] / [[concepts/retry]] / [[concepts/model-providers]] / [[concepts/progress-drafts]]
