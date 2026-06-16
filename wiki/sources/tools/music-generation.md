---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/music-generation
source_path: raw/docs/tools/music-generation.md
doc_section: tools
title: "音楽生成"
ingested: 2026-06-14
tags: [music-generation, media, tool, background-task, provider]
related:
  - "[[sources/tools/media-overview]]"
  - "[[concepts/tasks]]"
  - "[[concepts/session-tool]]"
---

# 音楽生成（解説）

> 原典: `raw/docs/tools/music-generation.md` ・ https://docs.openclaw.ai/ja-JP/tools/music-generation

## 一言まとめ

`music_generate` ツールで、エージェントが音楽/音声を生成する（プロバイダーは Google・MiniMax・ComfyUI）。**バックグラウンドタスク**として開始し、完成時にエージェントを再起動して添付する。

## 位置づけ

[[sources/tools/media-overview]] の生成ツールで、[[concepts/tasks]] のタスク台帳に乗る非同期ジョブ（[[sources/tools/video-generation]] と同型）。エージェントツールは [[concepts/session-tool]]。

## 仕組み・ふるまい

- セッション実行ではバックグラウンドタスクとして開始 → 台帳で追跡 → 準備完了でエージェント再起動 → 通知＋音声添付。グループ/チャンネルではメッセージツールで中継。
- 完了エージェントが非公開返信のみ書いた場合、生成メディアを付けてチャネル直送にフォールバック（その旨を明示警告）。

## 設定・使い方の要点

- プロバイダー設定が前提。進捗は `openclaw tasks list|show`（[[sources/automation/tasks]]）。

## 注意点・落とし穴

- 非同期のため即時返らない。可視配信のグループでは直接投稿せずメッセージツール経由。

## 用語と略称

- **`music_generate`** = 音楽/音声を生成するツール
- **バックグラウンドタスク** = 非同期で進む作業（[[concepts/tasks]]）
- **ComfyUI** = ワークフローベースの生成バックエンド

## 関連ページ

- [[sources/tools/media-overview]] / [[sources/tools/video-generation]] / [[sources/tools/image-generation]]
- [[concepts/tasks]] / [[concepts/session-tool]]
