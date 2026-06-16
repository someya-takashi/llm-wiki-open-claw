---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/apply-patch
source_path: raw/docs/tools/apply-patch.md
doc_section: tools
title: "apply_patch ツール"
ingested: 2026-06-14
tags: [apply-patch, file-edit, patch, multi-file, tool]
related:
  - "[[concepts/session-tool]]"
  - "[[sources/tools/diffs]]"
  - "[[sources/tools/exec]]"
---

# apply_patch ツール（解説）

> 原典: `raw/docs/tools/apply-patch.md` ・ https://docs.openclaw.ai/ja-JP/tools/apply-patch

## 一言まとめ

`apply_patch` は**構造化パッチ形式でファイル変更を適用**するツール。単一の `edit` では壊れやすい複数ファイル/複数ハンクの編集に向く。1 つの `input` 文字列に複数のファイル操作をまとめる。

## 位置づけ

[[concepts/session-tool]] のファイル編集ツール。変更内容を読み取り専用 diff にするのは [[sources/tools/diffs]]、シェル経由の変更は [[sources/tools/exec]]（exec を無効化しても apply_patch とは独立）。

## 仕組み・ふるまい

- 1 つ以上のファイル操作をラップした `input` 文字列を受け取り、構造化パッチとして適用。複数ファイル・複数ハンクをアトミックに扱いやすい。

## 設定・使い方の要点

- 細かい複数箇所の編集に `edit` を繰り返すより堅牢。ツールポリシー/サンドボックスの書き込み許可に従う。

## 用語と略称

- **`apply_patch`** = 構造化パッチでファイルを編集するツール
- **ハンク（hunk）** = パッチ内の変更ブロック
- **`edit`** = 単一箇所のファイル編集ツール

## 関連ページ

- [[concepts/session-tool]] / [[sources/tools/diffs]] / [[sources/tools/exec]]
- [[concepts/sandboxing]]
