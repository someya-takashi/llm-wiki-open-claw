---
type: concept
aliases: [Session pruning, セッションのプルーニング, 刈り込み]
tags: [session-pruning, context, cache, tool-results]
related:
  - "[[concepts/context]]"
  - "[[concepts/compaction]]"
  - "[[concepts/session]]"
sources:
  - "[[sources/concepts/session-pruning]]"
updated: 2026-06-14
---

# セッションのプルーニング

**プルーニング（刈り込み）**は、各 LLM 呼び出しの前に**古いツール結果**（exec・ファイル読み取り・検索結果）をコンテキストからトリミングする機構。会話テキストは書き換えず、**メモリ内のみ**で行う（ディスクのトランスクリプトは不変、完全履歴は常に残る）。

## なぜ重要か

長いセッションではツール出力が積み上がってコンテキストを膨らませ、コスト増と早すぎる [[concepts/compaction]] を招く。プルーニングは特に **Anthropic のプロンプトキャッシュ**で効く：キャッシュ TTL（既定 5 分）が切れると次リクエストで全プロンプトが再キャッシュされるため、書き込みサイズを減らすことがそのままコスト削減になる。Anthropic プロファイルでは自動有効、他プロバイダーは既定オフ（`agents.defaults.contextPruning`）。

## プルーニング vs Compaction

| | プルーニング | [[concepts/compaction]] |
| --- | --- | --- |
| 何をするか | ツール結果をトリム | 会話を要約 |
| 保存されるか | いいえ（リクエストごと） | はい（トランスクリプト内） |
| 対象 | ツール結果のみ | 会話全体 |

両者は相互補完的で、プルーニングは Compaction の合間にツール出力を軽量に保つ。手順（TTL 待ち→ソフトトリム/ハードクリア→TTL リセット）とレガシー画像クリーンアップは [[sources/concepts/session-pruning]]。

## 関連

- [[concepts/context]] / [[concepts/compaction]] / [[concepts/session]]
- [[concepts/context-engine]]
- 📝 ブログ（二次資料・外部）：直近 N ターンを残し古い履歴を捨てる「トリミング」技法（粒度はターン単位）と評価指針 → [[articles/context-engineering-session-memory]]
- 📝 ブログ（二次資料・Anthropic）：「ツール結果クリア」を最も安全で軽いコンパクション形と位置づける（OpenClaw のプルーニングに対応）→ [[articles/effective-context-engineering-for-ai-agents]]
