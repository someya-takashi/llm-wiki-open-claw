---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/image-generation
source_path: raw/docs/tools/image-generation.md
doc_section: tools
title: "画像生成"
ingested: 2026-06-14
tags: [image-generation, media, tool, provider, attachment]
related:
  - "[[sources/tools/media-overview]]"
  - "[[concepts/session-tool]]"
  - "[[concepts/agent-runtimes]]"
---

# 画像生成（解説）

> 原典: `raw/docs/tools/image-generation.md` ・ https://docs.openclaw.ai/ja-JP/tools/image-generation

## 一言まとめ

`image_generate` ツールで、エージェントが設定済みプロバイダーを使って**画像を生成・編集**する。生成画像は返信のメディア添付として自動配信される。

## 位置づけ

[[sources/tools/media-overview]] の生成ツールの一つ（[[concepts/session-tool]]）。プロバイダー選択は [[concepts/agent-runtimes]] の provider 層（`agents.defaults.imageGenerationModel`）。受信画像の理解は別物（[[concepts/media-understanding]] の `image` ツール）。

## 仕組み・ふるまい

- プロンプト（任意で参照画像）から画像を生成/編集し、返信に添付。プロバイダーは `google/*`・`openai/*`・`fal/*` 等。

## 設定・使い方の要点

- `agents.defaults.imageGenerationModel.primary`（例 `google/gemini-3-pro-image-preview`・`fal/fal-ai/flux/dev`）＋プロバイダー認証。ネイティブ画像分析は別の `image` ツール（`agents.defaults.imageModel`）。

## 注意点・落とし穴

- 標準の画像生成はコアの `image_generate` を使う（カスタムワークフローのみ skill）。プロバイダー API キーが要る。

## 用語と略称

- **`image_generate`** = 画像を作る/編集するツール
- **`image`** = 受信画像を分析する別ツール（理解側）
- **`imageGenerationModel` / `imageModel`** = 生成用 / 分析用のモデル設定

## 関連ページ

- [[sources/tools/media-overview]] / [[sources/tools/music-generation]] / [[sources/tools/video-generation]]
- [[concepts/session-tool]] / [[concepts/agent-runtimes]] / [[concepts/media-understanding]]
