---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/voice-call
source_path: raw/docs/plugins/voice-call.md
doc_section: plugins
title: "音声通話 Plugin"
ingested: 2026-06-14
tags: [plugin, voice-call, telephony, twilio, realtime, transcription, tts]
related:
  - "[[components/plugin-system]]"
  - "[[concepts/voice]]"
  - "[[components/gateway]]"
---

# 音声通話 Plugin（解説）

> 原典: `raw/docs/plugins/voice-call.md` ・ https://docs.openclaw.ai/ja-JP/plugins/voice-call

## 一言まとめ

エージェントに**電話通話**をさせる Plugin。アウトバウンド通知・マルチターン会話・全二重リアルタイム音声・ストリーミング文字起こし・許可リスト付きインバウンド通話をサポート。プロバイダーは Twilio・Telnyx・Plivo・mock（開発用）。

## 位置づけ

[[components/plugin-system]] の電話統合で、[[concepts/voice]] を電話網に拡張したもの（Talk がデバイス上の音声なら、こちらは PSTN/SIP 経由）。[[components/gateway]] プロセス内で実行。

## 仕組み・ふるまい

- **リアルタイム音声**：Google Gemini Live・OpenAI 等の双方向プロバイダー。**ストリーミング文字起こし**：OpenAI・xAI 等。**通話用 TTS**：コア TTS or ElevenLabs 上書き（[[sources/tools/tts]]）。
- **着信通話**：番号ごとルーティング・発話出力コントラクト・会話開始時の動作・古い通話リーパー。Webhook セキュリティ（署名検証）。

## 設定・使い方の要点

- 設定 `plugins.entries.voice-call`（プロバイダー認証情報・セッションスコープ・realtime/transcription/TTS の上書き）。CLI・エージェントツール・Gateway RPC を提供。
- Google Meet の Twilio 参加と連携（[[sources/plugins/google-meet]]）。

## 注意点・落とし穴

- ⚠️ Webhook 公開（インバウンド受信）・署名検証・プロバイダー認証情報が要設定。リモート Gateway では Gateway ホストにインストール・再起動。

## 用語と略称

- **全二重（full-duplex）** = 同時に話す/聞くリアルタイム音声
- **STT / TTS** = 文字起こし / 音声合成
- **PSTN / SIP** = 公衆電話網 / IP 電話プロトコル
- **Twilio / Telnyx / Plivo** = 電話 API プロバイダー
- **リーパー（reaper）** = 古い通話を回収する仕組み

## 関連ページ

- [[components/plugin-system]] / [[concepts/voice]] / [[components/gateway]]
- [[sources/plugins/google-meet]] / [[sources/tools/tts]]
