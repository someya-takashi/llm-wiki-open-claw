---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/media-understanding
source_path: raw/docs/nodes/media-understanding.md
doc_section: nodes
title: "メディア理解"
ingested: 2026-06-14
tags: [media-understanding, image, audio, video, transcription, tools-media, fallback]
related:
  - "[[concepts/media-understanding]]"
  - "[[concepts/messages]]"
  - "[[concepts/agent-runtimes]]"
---

# メディア理解（解説）

> 原典: `raw/docs/nodes/media-understanding.md` ・ https://docs.openclaw.ai/ja-JP/nodes/media-understanding

## 一言まとめ

受信した**画像/音声/動画を、返信パイプラインの前に短いテキストへ事前要約**する仕組み（`tools.media`）。ルーティングとコマンド解析を速く正確にしつつ、**元のメディアは常にモデルへも渡す**。

## 位置づけ

[[concepts/media-understanding]] の中核ソース。[[concepts/messages]] の受信処理の一段で、要約モデルは [[concepts/agent-runtimes]] の provider/CLI を使う。ベンダー固有の挙動はベンダー Plugin が登録し、core は共有の `tools.media` 設定・フォールバック順・パイプライン統合を所有。

## 仕組み・ふるまい

1. 添付収集（`MediaPaths`/`MediaUrls`/`MediaTypes`）→ 2. 機能（画像/音声/動画）ごとに添付選択（既定: 最初）→ 3. サイズ+機能+認証で最初の適格モデルを選択 → 4. 失敗/大きすぎたら**次のエントリにフォールバック** → 5. 成功で `Body` を `[Image]`/`[Audio]`/`[Video]` ブロックに、音声は `{{Transcript}}` を設定。
- **理解が失敗/無効でも返信は続行**（元の本文＋添付を使用）。プライマリ画像モデルが既にビジョン対応なら要約をスキップして元画像を渡す。
- モデルエントリは `type: "provider"`（provider/model/prompt/maxChars/capabilities/profile）または `type: "cli"`（command/args、`{{MediaPath}}`/`{{MaxChars}}` テンプレ）。

## 設定・使い方の要点

- `tools.media.models`（共有）＋ `tools.media.image|audio|video`（機能別上書き・`models`/`attachments`/`scope`）。`concurrency` 既定 2。
- 既定上限：画像 10MB・音声 20MB・動画 50MB。`maxChars` 画像/動画 500、音声は未設定（全文）。1024B 未満の音声は破損扱いでスキップ。
- **自動検出順**（モデル未設定・`enabled≠false`）：アクティブ返信モデル → `agents.defaults.imageModel`（画像）→ ローカル CLI（音声: sherpa-onnx/whisper-cli/whisper）→ Gemini CLI → プロバイダー認証（音声: OpenAI→Groq→xAI→Deepgram→…／画像: OpenAI→Anthropic→Google→…／動画: Google→Qwen→Moonshot）。

## 注意点・落とし穴

- ⚠️ **ベストエフォート**：エラーは返信をブロックしない。`scope` で実行場所を限定（例 DM のみ）。`/status` に `📎 Media: image ok (...)・audio skipped (maxBytes)` 行。
- 抽出ファイルテキストは `<<<EXTERNAL_UNTRUSTED_CONTENT>>>` 境界で「信頼されない外部データ」としてラップ（命令ではなくデータ扱い）。

## 用語と略称

- **メディア理解（media understanding）** = 受信メディアをテキスト要約する処理
- **STT** = Speech-to-Text（音声文字起こし）
- **`{{Transcript}}`** = 音声文字起こしを差すテンプレート変数
- **capabilities** = エントリが扱うメディア種別（image/audio/video）

## 関連ページ

- [[concepts/media-understanding]] — 対応する概念ページ
- [[sources/nodes/audio]] / [[sources/nodes/images]] / [[concepts/messages]]
- [[sources/gateway/cli-backends]] / [[concepts/security]]
