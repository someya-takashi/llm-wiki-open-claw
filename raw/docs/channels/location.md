---
title: "チャンネルlocation解析"
source: "https://docs.openclaw.ai/ja-JP/channels/location"
author:
published:
created: 2026-06-14
description: "受信チャンネルのlocation解析（Telegram/WhatsApp/Matrix）とコンテキストフィールド"
tags:
  - "clippings"
---
OpenClawは、チャットチャンネルから共有されたlocationを次の形式に正規化します。

- 受信本文に追加される簡潔な座標テキスト
- 自動返信コンテキストペイロード内の構造化フィールド

チャンネルが提供するラベル、住所、caption/commentは、ユーザー本文にインラインで入るのではなく、共有の信頼されていないメタデータJSONブロックとしてプロンプトに描画されます。

現在サポートされているもの:

- **Telegram** （locationピン + venue + ライブロケーション）
- **WhatsApp** （ `locationMessage` + `liveLocationMessage` ）
- **Matrix** （ `geo_uri` を持つ `m.location` ）

## テキスト形式

locationは、角括弧なしの見やすい行として描画されます。

- ピン:
	- `📍 48.858844, 2.294351 ±12m`
- 名前付きの場所:
	- `📍 48.858844, 2.294351 ±12m`
- ライブ共有:
	- `🛰 Live location: 48.858844, 2.294351 ±12m`

チャンネルにラベル、住所、またはcaption/commentが含まれている場合、それはコンテキストペイロードに保持され、フェンス付きの信頼されていないJSONとしてプロンプトに表示されます。

text

```
Location（信頼されていないメタデータ）:
\`\`\`json
{
  "latitude": 48.858844,
  "longitude": 2.294351,
  "name": "Eiffel Tower",
  "address": "Champ de Mars, Paris",
  "caption": "Meet here"
}
\`\`\`
```

## コンテキストフィールド

locationが存在する場合、これらのフィールドが `ctx` に追加されます。

- `LocationLat` （number）
- `LocationLon` （number）
- `LocationAccuracy` （number、メートル; 任意）
- `LocationName` （string; 任意）
- `LocationAddress` （string; 任意）
- `LocationSource` （ `pin | place | live` ）
- `LocationIsLive` （boolean）
- `LocationCaption` （string; 任意）

プロンプトレンダラーは `LocationName` 、 `LocationAddress` 、 `LocationCaption` を信頼されていないメタデータとして扱い、他のチャンネルコンテキストで使われるものと同じ境界付きJSONパスを通じてシリアライズします。

## チャンネルに関する注記

- **Telegram**: venueは `LocationName/LocationAddress` にマッピングされます。ライブロケーションは `live_period` を使用します。
- **WhatsApp**: `locationMessage.comment` と `liveLocationMessage.caption` は `LocationCaption` を設定します。
- **Matrix**: `geo_uri` はピンlocationとして解析されます。高度は無視され、 `LocationIsLive` は常にfalseです。

## 関連

- [Location command (nodes)](https://docs.openclaw.ai/ja-JP/nodes/location-command)
- [Camera capture](https://docs.openclaw.ai/ja-JP/nodes/camera)
- [Media understanding](https://docs.openclaw.ai/ja-JP/nodes/media-understanding)