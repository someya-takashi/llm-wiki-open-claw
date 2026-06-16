---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/talk
source_path: raw/docs/nodes/talk.md
doc_section: nodes
title: "トークモード"
ingested: 2026-06-14
tags: [voice, talk, realtime, stt-tts, transcription, conversation-loop]
related:
  - "[[concepts/voice]]"
  - "[[sources/tools/tts]]"
  - "[[sources/nodes/voicewake]]"
---

# トークモード（解説）

> 原典: `raw/docs/nodes/talk.md` ・ https://docs.openclaw.ai/ja-JP/nodes/talk

## 一言まとめ

**継続的な音声会話ループ**（聞く→文字起こしをモデルへ→応答→読み上げ）。ネイティブ（macOS/iOS/Android）とブラウザーの 2 形態があり、リアルタイム/STT-TTS/文字起こし専用のモードを持つ。

## 位置づけ

[[concepts/voice]] の中核ソース。出力（読み上げ）側は [[sources/tools/tts]]、起動のトリガーは [[sources/nodes/voicewake]]。ノードは `talk` capability を広告し `talk.*` コマンドを宣言（[[sources/nodes/nodes]]）。

## 仕組み・ふるまい

- **ネイティブ Talk**：①音声を聞く → ②アクティブセッション経由で文字起こしを送信 → ③応答を待つ → ④Talk プロバイダー（`talk.speak`）で読み上げ。macOS は常時オーバーレイ（Listening→Thinking→Speaking）、短い無音で送信、返信は WebChat に書かれる、**発話割り込み**既定オン。
- **ブラウザー Talk**：クライアント所有は `talk.client.create`（`webrtc`/`provider-websocket`）、Gateway 所有は `talk.session.create`（`gateway-relay`）。ツール呼び出しは `openclaw_agent_consult` へ `talk.client.toolCall` で転送（`chat.send` は直接呼ばない）。
- **文字起こし専用**：`mode: "transcription"`＋`brain: "none"`（キャプション/ディクテーション用、アシスタント音声なし）。
- **返信内の音声ディレクティブ**：返信先頭の単一 JSON 行 `{ "voice": "...", "once": true }` で音声を制御（`once` なしは新既定に。TTS 再生前に行は削除）。

## 設定・使い方の要点

- `talk` 配下：`provider`（elevenlabs/mlx/system）・`providers.<p>`（voiceId/modelId/outputFormat/apiKey）・`speechLocale`・`silenceTimeoutMs`（既定 macOS/Android 700ms・iOS 900ms）・`interruptOnSpeech`（既定 true）。
- `talk.realtime`：`provider`（webrtc=openai 等）・`voice`（marin/cedar 推奨）・`brain`（`agent-consult`=Gateway ポリシー経由／`direct-tools`／`none`）・`instructions`。`talk.catalog` が対応モード/トランスポート/voice を公開。

## 注意点・落とし穴

- Speech＋Microphone 権限が必要。Android は `talk.speak` RPC が無いときのみローカル TTS にフォールバック。`eleven_v3` の `stability` は 0.0/0.5/1.0 のみ。

## 用語と略称

- **STT-TTS** = 文字起こし＋読み上げで構成する Talk モード
- **realtime** = プロバイダーのリアルタイム音声 API を使う低遅延モード
- **WebRTC** = ブラウザーのリアルタイム音声/映像通信規格
- **brain** = Talk のツール/推論ルーティング戦略（agent-consult/direct-tools/none）
- **`openclaw_agent_consult`** = リアルタイム Talk から本体エージェント実行を呼ぶツール

## 関連ページ

- [[concepts/voice]] — 対応する概念ページ
- [[sources/tools/tts]] / [[sources/nodes/voicewake]] / [[sources/nodes/audio]] / [[sources/nodes/media-understanding]]
