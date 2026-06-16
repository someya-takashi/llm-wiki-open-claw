---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/announcements/bluebubbles-imessage
source_path: raw/docs/announcements/bluebubbles-imessage.md
doc_section: announcements
title: "BlueBubbles の削除と imsg の iMessage 経路"
ingested: 2026-06-14
tags: [announcement, imessage, bluebubbles, imsg, migration, channel]
related:
  - "[[concepts/channel-routing]]"
  - "[[sources/channels/channels]]"
---

# BlueBubbles の削除と imsg の iMessage 経路（解説）

> 原典: `raw/docs/announcements/bluebubbles-imessage.md` ・ https://docs.openclaw.ai/ja-JP/announcements/bluebubbles-imessage
>
> ℹ️ `announcements/` セクションの変更告知。

## 一言まとめ

OpenClaw は **BlueBubbles チャネルを同梱しなくなり**、iMessage は同梱 `imessage` Plugin（`imsg` ブリッジ）経由に移行した。`channels.bluebubbles` 設定は `channels.imessage` へ移行する。

## 位置づけ

[[concepts/channel-routing]] の iMessage チャネルの実装変更。チャネルカタログは [[sources/channels/channels]]。

## 仕組み・ふるまい

- 同梱 `imessage` Plugin が [`imsg`](https://github.com/steipete/imsg) をローカル or SSH ラッパー経由で起動し、stdin/stdout 上の JSON-RPC で通信する。
- 旧 `/channels/bluebubbles` ドキュメント URL は「BlueBubbles からの移行」にリダイレクト（設定変換表＋切り替えチェックリスト）。

## 設定・使い方の要点

- 既存 `channels.bluebubbles` を `channels.imessage` に移行。サインイン済み Mac でホスト権限と Messages アクセスがあれば、新規 iMessage セットアップで推奨。

## 注意点・落とし穴

- ⚠️ BlueBubbles は非同梱。古い設定は移行ガイドに従って変換する。

## 用語と略称

- **BlueBubbles** = 旧 iMessage 連携（非同梱になった）
- **imsg** = iMessage を JSON-RPC で操作する同梱ブリッジ
- **JSON-RPC** = JSON ベースのリモート手続き呼び出し

## 関連ページ

- [[concepts/channel-routing]] / [[sources/channels/channels]]
- [[channels/imessage]]（未作成）
