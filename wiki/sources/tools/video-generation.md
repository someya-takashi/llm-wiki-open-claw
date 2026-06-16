---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/video-generation
source_path: raw/docs/tools/video-generation.md
doc_section: tools
title: "動画生成"
ingested: 2026-06-14
tags: [video-generation, tool, async, tasks, providers, multimodal]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/queue]]"
  - "[[concepts/media-understanding]]"
---

# 動画生成（解説）

> 原典: `raw/docs/tools/video-generation.md` ・ https://docs.openclaw.ai/ja-JP/tools/video-generation

## 一言まとめ

エージェントが **テキスト/参照画像/既存動画から動画を生成**する `video_generate` ツール。16 プロバイダーバックエンド対応で、**非同期タスク**として実行され、完成時に同じセッションを起動して結果を届ける。

## 位置づけ

[[concepts/session-tool]] のエージェントツールの一つで、[[concepts/queue]]/タスク台帳に乗る非同期処理。受信メディアを読む [[concepts/media-understanding]] の「生成側カウンターパート」。TTS（[[sources/tools/tts]]）と並ぶメディア出力機能。

## 仕組み・ふるまい

- **3 ランタイムモード**：`generate`（テキスト→動画）/`imageToVideo`（参照画像あり）/`videoToVideo`（参照動画あり）。プロバイダーは任意のサブセットを対応し、送信前に検証（`action=list` で対応モード報告）。
- **非同期フロー**：呼び出し → 即座にタスク ID 返却 → プロバイダーがバックグラウンド処理（通常 30 秒〜数分）→ 完成で内部完了イベントが同セッションを起動 → エージェントが通知し動画を添付。実行中の重複呼び出しは現タスク状態を返す。セッション外（直接ツール呼び出し）はインライン生成にフォールバック。
- **タスクライフサイクル**：`queued`→`running`→`succeeded`/`failed`。CLI `openclaw tasks list|show|cancel <taskId>`。

## 設定・使い方の要点

- API キー設定で自動的にツールが現れる（許可リスト登録不要）。既定モデルは `agents.defaults.videoGenerationModel.primary`。
- 生成動画は OpenClaw 管理メディアストレージへ（上限は動画メディア制限、`agents.defaults.mediaMaxMb` で引き上げ）。ホスト URL を返すプロバイダーはサイズ超過時に URL 配信にフォールバック。
- プロバイダー例：Google（veo）・OpenAI（sora-2）・Runway（gen4.5）・MiniMax・BytePlus Seedance・fal・Alibaba/Qwen（wan）・xAI（grok-imagine-video）など。

## 注意点・落とし穴

- グループ/チャンネルでメッセージツールのみ可視配信の場合、エージェントが直接投稿せずメッセージツールで中継。`videoToVideo` は多くのプロバイダーでリモート `http(s)` URL を要するためスキップされることがある。

## 用語と略称

- **T2V / I2V / V2V** = Text/Image/Video-to-Video
- **タスク台帳（task ledger）** = 非同期ジョブを追跡する仕組み（`openclaw tasks`）
- **veo / sora / seedance** = 各社の動画生成モデル

## 関連ページ

- [[concepts/session-tool]] / [[concepts/queue]] / [[concepts/media-understanding]]
- [[sources/tools/tts]] / [[providers/google]]
