---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/google-meet
source_path: raw/docs/plugins/google-meet.md
doc_section: plugins
title: "Google Meet Plugin"
ingested: 2026-06-14
tags: [plugin, google-meet, meeting, voice, transcription, oauth, twilio]
related:
  - "[[components/plugin-system]]"
  - "[[concepts/voice]]"
  - "[[concepts/media-understanding]]"
---

# Google Meet Plugin（解説）

> 原典: `raw/docs/plugins/google-meet.md` ・ https://docs.openclaw.ai/ja-JP/plugins/google-meet

## 一言まとめ

エージェントを **Google Meet 会議に参加させる**明示的な Plugin。リアルタイム文字起こしが聞き取り、設定済み OpenClaw エージェントが回答し、TTS が Meet に音声を流す（既定 `agent` モード）。明示的な Meet URL にのみ参加し、自動の同意告知は無い。

## 位置づけ

[[components/plugin-system]] の大型統合で、[[concepts/voice]]（聞く/話す）と [[concepts/media-understanding]]（文字起こし）を会議に拡張したもの。ブラウザ参加には [[components/node]]（Chrome を持つノードホスト）を使う。

## 仕組み・ふるまい

- **モード**：`agent`（ライブ聞き取り→OpenClaw エージェント応答→TTS、既定）／`bidi`（直接リアルタイム音声モデルのフォールバック）／`transcribe`（応答ブリッジなしで参加・制御）。
- **トランスポート**：Chrome（既定の音声バックエンドは `BlackHole 2ch`）／Twilio（電話経由）。
- **認証**：個人の Google OAuth、またはサインイン済み Chrome プロファイル。Meet API で新しいスペースを作成して参加することも可能。

## 設定・使い方の要点

- OAuth：Google 認証情報を作成 → リフレッシュトークン発行 → `openclaw doctor` で検証。設定は `plugins.entries.google-meet`、ツールは Meet 参加/制御系。
- リモートカタログ制限・ライブテストチェックリスト・番号ごとのルーティング（Twilio）等は原典参照（大型ドキュメント）。

## 注意点・落とし穴

- ⚠️ **自動の同意告知が無い**ため、録音/参加の法的・倫理的配慮は運用者の責任。エージェントが参加するが話さない/会議作成失敗/Twilio 不通などのトラブルは原典のトラブルシューティング章に詳しい。

## 用語と略称

- **agent / bidi / transcribe** = 会議での応答モード（エージェント応答 / 直接リアルタイム音声 / 文字起こしのみ）
- **OAuth** = サードパーティ認証（Google ログイン）
- **TTS** = Text-to-Speech（音声合成、[[concepts/voice]]）
- **BlackHole 2ch** = macOS の仮想オーディオデバイス
- **Twilio** = 電話/メディアストリームのプロバイダー

## 関連ページ

- [[components/plugin-system]] / [[concepts/voice]] / [[concepts/media-understanding]]
- [[sources/plugins/voice-call]] / [[components/node]] / [[concepts/oauth]]
