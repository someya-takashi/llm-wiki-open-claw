---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/streaming
source_path: raw/docs/concepts/streaming.md
doc_section: concepts
title: "ストリーミングとチャンク化"
ingested: 2026-06-14
tags: [streaming, chunking, block-streaming, preview, tool-progress]
related:
  - "[[concepts/streaming]]"
  - "[[concepts/messages]]"
  - "[[concepts/progress-drafts]]"
---

# ストリーミングとチャンク化（解説）

> 原典: `raw/docs/concepts/streaming.md` ・ https://docs.openclaw.ai/ja-JP/concepts/streaming

## 一言まとめ

OpenClaw のストリーミングは **2 つの独立した層**――①**ブロックストリーミング**（完成した「ブロック」を通常のチャネルメッセージとして送出）と②**プレビューストリーミング**（生成中に一時的なプレビューを編集更新）。**真のトークン差分ストリーミングは無い**（プレビューはメッセージベースの送信＋編集/追記）。

## 位置づけ

[[concepts/messages]] パイプラインの「送信」段でのリアルタイム表示を担う。進捗ステータスに特化した `progress` モードは [[concepts/progress-drafts]] が詳説。配信失敗時の再送は [[concepts/retry]]。

## 仕組み・ふるまい

### ブロックストリーミング（チャネルメッセージ）

利用可能になったアシスタント出力を**粗いチャンク**で送る。`EmbeddedBlockChunker` が min/max 境界と区切り優先で分割：

- **下限**：バッファ ≥ `minChars` まで送出しない。**上限**：`maxChars` 前で分割（強制時は `maxChars` で）。区切り優先は `paragraph → newline → sentence → whitespace → ハード`。**コードフェンス内では分割しない**（強制時は閉じて再オープン）。`maxChars` はチャネルの `textChunkLimit` にクランプ。
- 制御：`blockStreamingDefault`（既定 off）、`blockStreamingBreak`（`text_end`＝chunker が送出次第フラッシュ / `message_end`＝メッセージ完了まで待つ）、`blockStreamingChunk`（min/max/breakPreference）、`blockStreamingCoalesce`（送信前にブロックを結合し「1 行スパム」を減らす、`idleMs` アイドル後にフラッシュ）。**設定は `agents.defaults` 配下**（ルートではない）。Telegram 以外は `*.blockStreaming: true` の明示が必要。
- **ブロック間の人間らしい間隔**：`humanDelay`（`off`/`natural` 800–2500ms/`custom`）。ブロック返信のみ。

### プレビューストリーミング（編集可能チャネル）

`channels.<channel>.streaming`（正規キー）。モード：`off` / `partial`（最新テキストで置換する単一プレビュー）/ `block`（チャンク化して追記更新）/ `progress`（進捗/ステータス、完了時に最終回答 → [[concepts/progress-drafts]]）。Telegram/Discord/Slack/Mattermost/Matrix/Teams が対応（細部は差あり。Slack は `nativeTransport` でネイティブストリーミング API、Teams はネイティブ進捗）。**`streaming.mode: "block"` はプレビューであって通常のブロック配信ではない**（通常ブロック返信は `streaming.block.enabled` かレガシー `blockStreaming` キー）。

### ツール進捗プレビュー

「Web を検索中」等の短いステータス行を、ツール実行中に同じプレビュー内へ表示（Discord/Slack/Telegram/Matrix が既定対応）。`streaming.preview.toolProgress: false` で非表示、`commandText: "status"` で生コマンドを隠す（既定 `"raw"`）。

## 設定・使い方の要点

- 「チャンクをストリーミング」：`blockStreamingDefault: "on"` ＋ `blockStreamingBreak: "text_end"`（＋ Telegram 以外は `*.blockStreaming: true`）。「最後にまとめて」：`message_end`。「無し」：`off`（最終返信のみ）。
- 結合の既定 `minChars` は Signal/Slack/Discord で 1500 に引き上げ。Discord は `maxLinesPerMessage`（既定 17）で縦長返信を分割。

## 注意点・落とし穴

- **二重ストリーミング回避**：ブロックストリーミングが明示有効ならプレビューはスキップされる。
- **メディア重複の抑制**：`MEDIA:` を早期ストリームし、最終ペイロードも同じ URL を含む場合、最終配信は重複メディアを取り除く（音声メモ/ファイルの二重送信防止）。
- レガシーキー（`streamMode`・スカラー `streaming`・`nativeStreaming` 等）は `openclaw doctor --fix` で `streaming.*` へ移行。

## 用語と略称

- **ブロックストリーミング** = 完成ブロックを通常メッセージとして送る層
- **プレビューストリーミング** = 一時メッセージを編集更新する層
- **チャンク化（chunking）** = チャネルの文字数上限に合わせて分割すること
- **coalesce（結合）** = 連続ブロックをまとめて送る処理
- **Block Kit** = Slack の構造化リッチ表示形式

## 関連ページ

- [[concepts/streaming]] — 対応する概念ページ
- [[concepts/messages]] / [[concepts/progress-drafts]] / [[concepts/retry]]
- [[sources/concepts/message-lifecycle-refactor]]
