---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/audio
source_path: raw/docs/nodes/audio.md
doc_section: nodes
title: "音声とボイスメモ"
ingested: 2026-06-14
tags: [audio, voice-memo, transcription, media-understanding, mention-gating]
related:
  - "[[concepts/media-understanding]]"
  - "[[sources/nodes/media-understanding]]"
  - "[[concepts/messages]]"
---

# 音声とボイスメモ（解説）

> 原典: `raw/docs/nodes/audio.md` ・ https://docs.openclaw.ai/ja-JP/nodes/audio

## 一言まとめ

受信した**ボイスメモ/音声を文字起こし**して返信パイプラインに載せる、メディア理解の音声特化版（`tools.media.audio`）。文字起こしは `{{Transcript}}` になり、スラッシュコマンド解析やグループのメンション判定にも使われる。

## 位置づけ

[[concepts/media-understanding]] の音声面（全体像は [[sources/nodes/media-understanding]]）。[[concepts/messages]] のメンションゲートと密接で、音声でも `@Bot` を拾えるようにする。

## 仕組み・ふるまい

- 流れ：最初の音声添付を取得 → `maxBytes`（既定 20MB）適用 → 適格モデルを順に実行（provider/CLI）→ 失敗/スキップで次へ → 成功で `[Audio]` ブロック＋`{{Transcript}}` 設定。`CommandBody`/`RawBody` も文字起こしに設定（コマンドが動くように）。
- **自動検出**：アクティブ返信モデル → ローカル CLI（sherpa-onnx-offline/whisper-cli/whisper）→ Gemini CLI → プロバイダー（OpenAI→Groq→xAI→Deepgram→Google→SenseAudio→ElevenLabs→Mistral）。
- **グループのメンション preflight**：`requireMention: true` のグループでは、音声をメンション確認の**前に**文字起こしし、ボイスメモ内の `@Bot` を検出して通す（失敗時はテキストのみ判定にフォールバック）。

## 設定・使い方の要点

- `tools.media.audio`：`models`（provider/CLI）・`maxBytes`・`maxChars`（既定未設定＝全文）・`scope`・`attachments`（`mode: "all"`＋`maxAttachments`）。`echoTranscript`（既定 false）＋`echoFormat` で文字起こしを発信元へエコー。
- Telegram は `channels.telegram.groups.<chatId>.disableAudioPreflight` で preflight をグループ/トピック単位に無効化。

## 注意点・落とし穴

- ⚠️ CLI は終了コード 0＋プレーンテキスト出力が前提（JSON は `jq -r .text` 等で整形）。CLI stdout 上限 5MB。`{{MediaPath}}` を args に使う。
- 1024B 未満の極小音声はスキップ。送信プロキシ環境変数（`HTTPS_PROXY` 等）を尊重。

## 用語と略称

- **STT** = Speech-to-Text（文字起こし）
- **preflight 文字起こし** = メンション判定の前に行う先行文字起こし
- **Whisper / Deepgram / Voxtral** = 文字起こしモデル/プロバイダー
- **ボイスメモ（voice memo）** = チャットの音声メッセージ

## 関連ページ

- [[concepts/media-understanding]] / [[sources/nodes/media-understanding]]
- [[sources/nodes/talk]] / [[sources/nodes/voicewake]] / [[concepts/messages]]
