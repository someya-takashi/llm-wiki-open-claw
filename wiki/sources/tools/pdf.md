---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/pdf
source_path: raw/docs/tools/pdf.md
doc_section: tools
title: "PDF ツール"
ingested: 2026-06-14
tags: [pdf, tool, parsing, text-extraction, media]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/media-understanding]]"
  - "[[sources/tools/media-overview]]"
---

# PDF ツール（解説）

> 原典: `raw/docs/tools/pdf.md` ・ https://docs.openclaw.ai/ja-JP/tools/pdf

## 一言まとめ

`pdf` は **1 つ以上の PDF を解析してテキストを返す**ツール。テキストが取れない PDF は先頭ページを画像化してモデルへ渡す（[[concepts/media-understanding]] と同じ `document-extract` 系の挙動）。

## 位置づけ

[[concepts/session-tool]] のファイル/メディアツール。受信メディア理解（[[concepts/media-understanding]]）の PDF 解析と同じ基盤で、[[sources/tools/media-overview]] の地図上にある。

## 仕組み・ふるまい

- 入力 PDF を解析してテキスト抽出。十分なテキストが無ければページを画像にラスタライズしてビジョンモデルへ。複数 PDF に対応。

## 設定・使い方の要点

- バンドルの `document-extract` Plugin（`pdfjs-dist` レガシービルド）を使う。HTTP API のファイル入力（[[sources/gateway/openresponses-http-api]]）でも同じ解析。

## 注意点・落とし穴

- 画像化された PDF は OCR ではなくビジョンモデルが読む。大きい PDF はサイズ/ページ上限に注意。

## 用語と略称

- **`pdf`** = PDF を解析してテキスト化するツール
- **ラスタライズ** = ページを画像に変換すること
- **document-extract** = PDF/文書解析の同梱 Plugin

## 関連ページ

- [[concepts/session-tool]] / [[concepts/media-understanding]] / [[sources/tools/media-overview]]
- [[sources/gateway/openresponses-http-api]]
