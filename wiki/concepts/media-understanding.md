---
type: concept
aliases: [Media Understanding, メディア理解, tools.media]
tags: [media, image, audio, video, transcription, multimodal, inbound]
related:
  - "[[concepts/messages]]"
  - "[[concepts/voice]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/context-engine]]"
components:
  - "[[components/node]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/nodes/media-understanding]]"
  - "[[sources/nodes/audio]]"
  - "[[sources/nodes/images]]"
updated: 2026-06-14
---

# メディア理解（Media Understanding）

メディア理解（media understanding, 受信した画像/音声/動画をテキストへ事前要約する処理）は、返信パイプラインが走る**前に**受信メディアを短いテキスト（`[Image]`/`[Audio]`/`[Video]` ブロック、音声は `{{Transcript}}`）に落とし込む仕組み。OpenClaw 共通の `tools.media` 設定が司り、ベンダー固有の挙動はベンダー Plugin が登録する。

## なぜ重要か

メッセージング基盤のエージェントには「写真だけ」「ボイスメモだけ」が届く。メディア理解があると、**ルーティングとコマンド解析（スラッシュコマンド/メンション判定）をテキストとして速く正確に**行え、しかも**元のメディアは常にモデルへも渡す**ので情報を失わない。これは [[concepts/messages]] の受信処理を「マルチモーダル対応」にする土台。

## 3 つの方向のうちの「入力側」

OpenClaw のメディアは 3 方向で捉えると整理しやすい：

| 方向 | 何をするか | 概念/ソース |
|---|---|---|
| **入力（理解）** | 画像/音声/動画 → テキスト要約 | **本ページ** |
| 出力（生成・音声） | テキスト → 音声/動画 | [[concepts/voice]]（TTS）・[[sources/tools/video-generation]] |
| リアルタイム音声 | 聞く↔話すの会話ループ | [[concepts/voice]]（Talk） |

## 仕組み（要点）

1. 添付収集 → 2. 機能（画像/音声/動画）ごとに添付選択 → 3. サイズ＋機能＋認証で最初の適格モデル選択 → 4. 失敗/サイズ超過なら**次のエントリにフォールバック**（順序付き）→ 5. 成功でブロック挿入。**理解が失敗/無効でも返信は続行**（元本文＋添付）。

モデルエントリは **provider 型**（OpenAI/Anthropic/Google/Deepgram…）または **cli 型**（whisper/gemini…、[[sources/gateway/cli-backends]] と同じ「ローカル CLI フォールバック」思想）。自動検出はアクティブ返信モデル→ローカル CLI→Gemini CLI→プロバイダー認証の順。詳細・設定キーは [[sources/nodes/media-understanding]]。

音声の特殊ケース（文字起こし、グループのメンション preflight）は [[sources/nodes/audio]]、送受信のサイズ規則・受信テンプレ変数（`{{MediaPath}}`/`{{MediaUrl}}`）は [[sources/nodes/images]]。

## 既存 wiki とのつながり

理解結果は [[concepts/context-engine]] が組み立てる文脈の一部になり、要約モデルの選択は [[concepts/agent-runtimes]] の provider/model 層に乗る。⚠️ 抽出ファイルテキストは `<<<EXTERNAL_UNTRUSTED_CONTENT>>>` 境界で「信頼されない外部データ」としてラップされる（[[concepts/threat-model]] の間接プロンプトインジェクション対策）。`scope` で実行場所（例 DM のみ）を絞れる。

## 代表ソース

- [[sources/nodes/media-understanding]] — `tools.media` の全体（モデル選択・フォールバック・自動検出）
- [[sources/nodes/audio]] — 音声文字起こしとメンション preflight
- [[sources/nodes/images]] — メディア送受信のサイズ規則と受信テンプレ変数

## 関連ページ

- [[concepts/messages]] / [[concepts/voice]] / [[concepts/agent-runtimes]] / [[concepts/context-engine]]
- [[components/node]] / [[components/gateway]] / [[concepts/threat-model]]
