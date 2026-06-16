---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/tts
source_path: raw/docs/tools/tts.md
doc_section: tools
title: "テキスト読み上げ（TTS）"
ingested: 2026-06-14
tags: [tts, voice, persona, providers, voice-message, talk]
related:
  - "[[concepts/voice]]"
  - "[[sources/nodes/talk]]"
  - "[[concepts/messages]]"
---

# テキスト読み上げ（TTS）（解説）

> 原典: `raw/docs/tools/tts.md` ・ https://docs.openclaw.ai/ja-JP/tools/tts

## 一言まとめ

送信する返信を **14 の音声プロバイダー**で音声化する仕組み（TTS = Text-to-Speech, 文章を音声に変換）。Feishu/Matrix/Telegram/WhatsApp ではネイティブ音声メッセージ、他では音声添付、電話/Talk には PCM/Ulaw ストリームとして配信。**既定はオフ**。

## 位置づけ

[[concepts/voice]] の出力側。Talk の `stt-tts` モードの読み上げを担い（[[sources/nodes/talk]]）、チャネル返信の音声化としても単独で機能する（[[concepts/messages]]）。`realtime` Talk はこの TTS パスでなくプロバイダー内で合成。

## 仕組み・ふるまい

- **プロバイダー**（14）：Azure Speech・DeepInfra・ElevenLabs・Google Gemini・Gradium・Inworld・Local CLI・Microsoft（Edge neural、キー不要・SLA なし）・MiniMax・OpenAI・OpenRouter・Volcengine・Vydra・xAI・Xiaomi MiMo。複数設定時は選択が最初、他はフォールバック。
- **ペルソナ**：プロバイダー非依存のプロンプトで声のトーン/話速等を定義し、各プロバイダーが自分の方式で反映（`promptTemplate`/`instructions` 等）。フォールバックポリシーあり。
- **モデルによるディレクティブ**：返信先頭の JSON 行で音声/速度などを上書き（[[sources/nodes/talk]] の音声ディレクティブと同形）。
- **出力**：固定フォーマット（チャネル別に最適化、音声ノート対応プロバイダーは Opus/Ogg）。Auto-TTS の挙動・チャンネル別出力形式あり。

## 設定・使い方の要点

- 有効化：`messages.tts.auto: "always"`＋`messages.tts.provider`（例 `elevenlabs`）。プロバイダーブロックは `messages.tts.providers.<id>`（apiKey/voice/model/outputFormat 等）。エージェントごとの音声上書きも可。
- スラッシュコマンド：`/tts status`・`/tts audio <text>`（単発音声返信）。ユーザーごと設定あり。エージェントツール `tts` は明示意図時のみ。Gateway RPC も提供。

## 注意点・落とし穴

- ⚠️ Microsoft プロバイダーは公開 Edge サービス（SLA/クォータなし、ベストエフォート）。レガシー ID `edge`→`microsoft` に正規化（`openclaw doctor --fix`）。
- 自動要約を使う場合は `summaryModel`（または既定モデル）のプロバイダーも認証が必要。

## 用語と略称

- **TTS** = Text-to-Speech（音声合成）
- **PCM / Ulaw** = 電話/Talk 向けの生音声ストリーム形式
- **ペルソナ（persona）** = 声のキャラクター（トーン/話速/スタイル）
- **Opus / Ogg** = 音声ノートの圧縮フォーマット
- **Auto-TTS** = 返信を自動で音声化するモード

## 関連ページ

- [[concepts/voice]] — 対応する概念ページ
- [[sources/nodes/talk]] / [[sources/nodes/voicewake]] / [[concepts/messages]]
- [[sources/tools/video-generation]] / [[providers/elevenlabs]]（未作成）
