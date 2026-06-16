---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/tokenjuice
source_path: raw/docs/tools/tokenjuice.md
doc_section: tools
title: "Tokenjuice"
ingested: 2026-06-14
tags: [tokenjuice, plugin, compaction, tool-result, exec-output]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/context]]"
  - "[[components/plugin-system]]"
---

# Tokenjuice（解説）

> 原典: `raw/docs/tools/tokenjuice.md` ・ https://docs.openclaw.ai/ja-JP/tools/tokenjuice

## 一言まとめ

`tokenjuice` は、コマンド実行**後**にノイズの多い `exec`/`bash` のツール結果をコンパクト化するオプションのバンドル Plugin。返される `tool_result` だけを縮め、コマンドや終了コードは変えない。

## 位置づけ

[[concepts/session-tool]] のツール結果に作用する [[components/plugin-system]] の Plugin で、[[concepts/context]] のトークン節約に効く（[[concepts/exec]] の冗長出力対策）。

## 仕組み・ふるまい

- `tool_result` を圧縮するのみ。シェル入力の書き換え・再実行・終了コード変更はしない。

## 設定・使い方の要点

- バンドル Plugin として有効化。冗長な exec 出力でコンテキストを浪費する運用に効く。

## 注意点・落とし穴

- 縮約は結果テキストのみ——元コマンドの意味は保持される。

## 用語と略称

- **Tokenjuice** = ツール結果を圧縮する Plugin
- **`tool_result`** = ツール実行の返り値
- **コンパクト化** = 冗長な出力を縮める処理

## 関連ページ

- [[concepts/session-tool]] / [[concepts/context]] / [[concepts/exec]]
- [[components/plugin-system]] / [[concepts/compaction]]
