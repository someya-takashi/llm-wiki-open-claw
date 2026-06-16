---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/location-command
source_path: raw/docs/nodes/location-command.md
doc_section: nodes
title: "位置情報コマンド"
ingested: 2026-06-14
tags: [node, location, gps, permissions, privacy]
related:
  - "[[components/node]]"
  - "[[sources/nodes/nodes]]"
  - "[[concepts/security]]"
---

# 位置情報コマンド（解説）

> 原典: `raw/docs/nodes/location-command.md` ・ https://docs.openclaw.ai/ja-JP/nodes/location-command

## 一言まとめ

ノードの `location.get`（`node.invoke` 経由）で**端末の現在地（緯度経度・精度・標高など）を取得**するコマンド。プライバシー配慮で**既定オフ**、アプリ内セレクター（オフ / 使用中のみ）＋「正確な位置情報」トグルで制御。

## 位置づけ

[[components/node]] の位置情報コマンド詳細（[[sources/nodes/nodes]]）。OS 権限が多段なので、アプリの UI セレクターは「要求モード」を決め、実際の許可は OS が決める。

## 仕組み・ふるまい

- 設定モデル（デバイスごと）：`location.enabledMode: off|whileUsing`、`location.preciseEnabled: bool`。
- `location.get` パラメーター：`timeoutMs`/`maxAgeMs`/`desiredAccuracy: coarse|balanced|precise`。レスポンスは `lat`/`lon`/`accuracyMeters`/`altitudeMeters`/`speedMps`/`headingDeg`/`timestamp`/`isPrecise`/`source(gps|wifi|cell)`。
- ツール統合：`nodes` ツールの `location_get` アクション、CLI `openclaw nodes location get --node <id>`。

## 設定・使い方の要点

- iOS/macOS は「使用中のみ/常に許可」、Android はフォアグラウンド位置情報のみ。正確な位置は別権限（iOS14+「正確」、Android fine/coarse）。
- エージェントは「ユーザーが有効化しスコープを理解している場合のみ」呼ぶガイドライン。

## 注意点・落とし穴

- 安定エラーコード：`LOCATION_DISABLED`（セレクターオフ）/`LOCATION_PERMISSION_REQUIRED`/`LOCATION_BACKGROUND_UNAVAILABLE`（背景で「使用中のみ」）/`LOCATION_TIMEOUT`/`LOCATION_UNAVAILABLE`。Android は背景の `location.get` を拒否（アプリを開いたままに）。

## 用語と略称

- **GPS** = Global Positioning System（衛星測位）
- **coarse / precise** = おおよそ / 正確な測位精度
- **whileUsing** = アプリ使用中のみ位置取得を許可するモード

## 関連ページ

- [[components/node]] / [[sources/nodes/nodes]] / [[sources/nodes/camera]]
- [[concepts/security]] / [[sources/channels/location]]（チャネル側の位置共有解析）
