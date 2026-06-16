---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/location
source_path: raw/docs/channels/location.md
doc_section: channels
title: "チャンネル location 解析"
ingested: 2026-06-14
tags: [location, channels, parsing, coordinates, context-field]
related:
  - "[[concepts/channel-routing]]"
  - "[[concepts/messages]]"
  - "[[sources/nodes/location-command]]"
---

# チャンネル location 解析（解説）

> 原典: `raw/docs/channels/location.md` ・ https://docs.openclaw.ai/ja-JP/channels/location

## 一言まとめ

チャットチャネルで**共有された位置情報**を正規化する仕組み。受信本文に簡潔な座標テキストを追加し、自動返信コンテキストに構造化フィールドを入れる。

## 位置づけ

[[concepts/channel-routing]] の受信処理（[[concepts/messages]]）で、ユーザーがチャットで送った位置を扱う。ノードが**取得する** `location.get`（[[sources/nodes/location-command]]）とは逆向き（ユーザー→エージェント）。

## 仕組み・ふるまい

- 受信した位置共有を①本文への簡潔な座標テキスト、②コンテキストペイロードの構造化フィールド（緯度/経度等）に正規化。チャネルごとに位置共有の形式が異なる。

## 設定・使い方の要点

- 受信メッセージのテンプレート/コンテキストで位置フィールドを参照できる。チャネル注記あり（対応状況がチャネル依存）。

## 用語と略称

- **location 共有** = チャットで送る位置情報
- **コンテキストフィールド** = 自動返信に渡る構造化データ
- **座標テキスト** = 緯度経度の簡潔な文字列表現

## 関連ページ

- [[concepts/channel-routing]] / [[concepts/messages]]
- [[sources/nodes/location-command]]（ノードが位置を取得する側）
