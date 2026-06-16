---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/media-overview
source_path: raw/docs/tools/media-overview.md
doc_section: tools
title: "メディアの概要"
ingested: 2026-06-14
tags: [media, image, video, music, tts, understanding, talk, overview]
related:
  - "[[concepts/media-understanding]]"
  - "[[concepts/voice]]"
  - "[[concepts/session-tool]]"
---

# メディアの概要（解説）

> 原典: `raw/docs/tools/media-overview.md` ・ https://docs.openclaw.ai/ja-JP/tools/media-overview
>
> ℹ️ メディア機能の地図。OpenClaw の「生成・理解・音声」の各ツールへ分岐する。

## 一言まとめ

OpenClaw のメディア機能は**すべてツール駆動**——画像/動画/音楽の**生成**、受信メディアの**理解**、返信の**音声化（TTS）**。各ツールはバックエンドプロバイダーが 1 つ以上設定されたときだけ現れる。

## 位置づけ

[[concepts/session-tool]] のメディア系ツールの入口。**入力（理解）**は [[concepts/media-understanding]]、**ライブ音声**は [[concepts/voice]]（Talk）、**生成・出力**は本ページが束ねる個別ツール。

## 仕組み・ふるまい

- **生成**：[[sources/tools/image-generation]]（`image_generate`）・[[sources/tools/music-generation]]（`music_generate`）・[[sources/tools/video-generation]]（`video_generate`）。生成物は返信にメディア添付として配信、音楽/動画は [[concepts/tasks]] のバックグラウンドタスク。
- **理解**：受信画像/音声/動画をテキスト要約（[[concepts/media-understanding]]）、PDF は [[sources/tools/pdf]]。
- **ライブ音声**：単発メディアツールでなく Talk セッション契約（`realtime`/`stt-tts`/`transcription`、[[concepts/voice]]）。telephony/会議/ブラウザ/PTT がカタログ・イベント・キャンセルを共有。

## 設定・使い方の要点

- 各ツールはプロバイダー設定が前提（例 `agents.defaults.imageGenerationModel`）。エージェントが会話に応じて使用タイミングを判断。

## 用語と略称

- **生成 / 理解 / 音声** = 作る / 読み取る / 話す・聞くの 3 方向
- **TTS / STT** = Text-to-Speech / Speech-to-Text
- **PTT** = Push-To-Talk
- **Talk セッション契約** = ライブ音声の共通プロトコル（[[concepts/voice]]）

## 関連ページ

- [[concepts/media-understanding]] / [[concepts/voice]] / [[concepts/session-tool]]
- [[sources/tools/image-generation]] / [[sources/tools/music-generation]] / [[sources/tools/video-generation]] / [[sources/tools/pdf]]
