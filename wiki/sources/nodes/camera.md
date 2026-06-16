---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/camera
source_path: raw/docs/nodes/camera.md
doc_section: nodes
title: "カメラキャプチャ"
ingested: 2026-06-14
tags: [node, camera, photo, video-clip, screen-record, permissions]
related:
  - "[[components/node]]"
  - "[[sources/nodes/nodes]]"
  - "[[concepts/media-understanding]]"
---

# カメラキャプチャ（解説）

> 原典: `raw/docs/nodes/camera.md` ・ https://docs.openclaw.ai/ja-JP/nodes/camera

## 一言まとめ

ノード（iOS/Android/macOS アプリ）の**カメラで写真（jpg）や短い動画クリップ（mp4）を撮る** `camera.*` コマンド。エージェントワークフローに「今この場の映像」を渡すための入口。

## 位置づけ

[[components/node]] のカメラ系コマンド詳細（[[sources/nodes/nodes]] の `camera.*` を掘り下げる）。撮ったメディアは [[concepts/media-understanding]] で要約され得る。

## 仕組み・ふるまい

- `camera.list`（デバイス一覧）/ `camera.snap`（`facing: front|back`・`maxWidth`・`quality`・`format: jpg`。写真は base64 を 5MB 未満に再圧縮）/ `camera.clip`（`durationMs` 既定 3000・最大 60000、`includeAudio` 既定 true、`mp4`）。
- **CLI ヘルパー**：`openclaw nodes camera snap/clip --node <id>`（一時ファイルに書き、`MEDIA:<path>` を出力。`snap` は既定で前後両方＝2 行）。
- *画面*動画（カメラでない）は `openclaw nodes screen record`（macOS は画面収録 TCC 権限）。

## 設定・使い方の要点

- ユーザー設定：iOS/Android は `camera.enabled` 既定オン、macOS アプリは `openclaw.cameraEnabled` 既定**オフ**（Settings → General → Allow Camera）。
- Android はランタイム権限 `CAMERA`（音声クリップに `RECORD_AUDIO`）。macOS の snap は `delayMs`（既定 2000ms）露出安定待ち。

## 注意点・落とし穴

- ⚠️ **フォアグラウンド必須**（背景は `NODE_BACKGROUND_UNAVAILABLE`）。無効時 `CAMERA_DISABLED`、権限なしは `*_PERMISSION_REQUIRED`。
- 動画クリップは base64 肥大回避のため ≤60s。OS の権限プロンプト＋ Info.plist 用途文字列が必要。

## 用語と略称

- **facing** = 前面/背面カメラの指定
- **MEDIA:<path>** = CLI が出力する一時メディアファイル参照
- **TCC** = macOS の透明性・同意・制御（画面収録/カメラ権限）

## 関連ページ

- [[components/node]] / [[sources/nodes/nodes]]
- [[sources/nodes/images]] / [[sources/nodes/media-understanding]] / [[sources/nodes/location-command]]
