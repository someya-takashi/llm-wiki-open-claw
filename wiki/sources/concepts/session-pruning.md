---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/session-pruning
source_path: raw/docs/concepts/session-pruning.md
doc_section: concepts
title: "セッションのプルーニング"
ingested: 2026-06-14
tags: [session-pruning, context, cache, compaction, tool-results]
related:
  - "[[concepts/session-pruning]]"
  - "[[concepts/context]]"
  - "[[concepts/compaction]]"
---

# セッションのプルーニング（解説）

> 原典: `raw/docs/concepts/session-pruning.md` ・ https://docs.openclaw.ai/ja-JP/concepts/session-pruning

## 一言まとめ

セッションのプルーニング（pruning, 刈り込み）は、各 LLM 呼び出しの前に**古いツール結果**（exec 結果・ファイル読み取り・検索結果）をコンテキストからトリミングし、通常の会話テキストは書き換えずにコンテキストの肥大を抑える仕組み。**メモリ内のみで行い、ディスク上のトランスクリプトは変更しない**。

## 位置づけ

[[concepts/context]] を軽く保つための機構の 1 つ。[[concepts/compaction]]（会話の要約圧縮）とは別物で**相互補完的**――プルーニングは Compaction サイクルの間ツール出力を軽量に保つ。設定は `agents.defaults.contextPruning.*`。

## 仕組み・ふるまい

1. プロンプトキャッシュの **TTL（Time To Live, 有効期限。既定 5 分）** が切れるのを待つ。
2. 古いツール結果を見つける（会話テキストは残す）。
3. 大きい結果は**ソフトトリム**（先頭と末尾を残し `...` を挿入）。
4. それ以外は**ハードクリア**（プレースホルダーに置換）。
5. TTL をリセットし、後続リクエストで新キャッシュを再利用。

### なぜ効くか（Anthropic キャッシュ）

長いセッションではツール出力が蓄積してコストが増え、必要以上に早く Compaction が要る。プルーニングは特に **Anthropic のプロンプトキャッシュ**で効果的：キャッシュ TTL が切れると次リクエストで全プロンプトが再キャッシュされるため、プルーニングがキャッシュ書き込みサイズを減らしコストを直接下げる。

### レガシー画像クリーンアップ（別機構）

履歴に残る生の画像ブロックや古いメディア参照を、冪等なリプレイビューで `[image data removed - already processed by model]` 等に置換する。直近 3 ターンはバイト単位で保持しキャッシュプレフィックスを安定させ、現在ターンの添付は維持して vision モデルが新規画像を扱えるようにする。生トランスクリプトは書き換えない。

## 設定・使い方の要点

- **賢いデフォルト**：Anthropic プロファイルでは自動有効（OAuth/トークン認証は Heartbeat 1 時間、API キーは 30 分）。明示値があれば上書きしない。
- Anthropic 以外は既定で無効。有効化：`agents.defaults.contextPruning: { mode: "cache-ttl", ttl: "5m" }`。無効化は `mode: "off"`。

## 注意点・落とし穴

- プルーニングは**保存されない**（リクエストごと）。完全な履歴は常にディスクに残るので、履歴ビューアーでは元のメッセージと画像を確認できる。

## 用語と略称

- **プルーニング（pruning, 刈り込み）** = メモリ内プロンプトから古いツール結果を落とす処理
- **TTL** = Time To Live（キャッシュの有効期限）
- **ソフトトリム / ハードクリア** = 一部を残して省略 / 丸ごとプレースホルダー化

## 関連ページ

- [[concepts/session-pruning]] — 対応する概念ページ
- [[concepts/context]] / [[concepts/compaction]] / [[concepts/session]]
- [[concepts/context-engine]]
