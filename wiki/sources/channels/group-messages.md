---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/group-messages
source_path: raw/docs/channels/group-messages.md
doc_section: channels
title: "WhatsApp グループメッセージ"
ingested: 2026-06-14
tags: [whatsapp, groups, group-allowlist, session-key, pending-context]
related:
  - "[[concepts/groups]]"
  - "[[channels/whatsapp]]"
  - "[[sources/channels/groups]]"
---

# WhatsApp グループメッセージ（解説）

> 原典: `raw/docs/channels/group-messages.md` ・ https://docs.openclaw.ai/ja-JP/channels/group-messages

## 一言まとめ

クロスチャネルのグループモデル（[[sources/channels/groups]]）の上に乗る **WhatsApp 固有のグループ挙動**——有効化・グループ許可リスト・グループごとのセッションキー・保留メッセージのコンテキスト注入。

## 位置づけ

[[concepts/groups]] の WhatsApp 実装。チャネル本体は [[channels/whatsapp]]。「WhatsApp グループに常駐し、ping されたときだけ起動、個人 DM とは別スレッド」を実現する。

## 仕組み・ふるまい

- グループ許可リストで応答対象を絞り、メンション時のみ起動。グループごとのセッションキーで個人 DM と分離。
- 保留メッセージ（メンション前の文脈）をコンテキストとして注入できる。

## 設定・使い方の要点

- `channels.whatsapp.groups.*`（有効化・許可リスト）。テスト/検証手順あり。

## 注意点・落とし穴

- WhatsApp は単一セッション（[[channels/whatsapp]]）なので、グループ常駐はそのアカウントが参加しているグループに限る。

## 用語と略称

- **グループ許可リスト** = 応答するグループの指定
- **グループセッションキー** = 個人 DM と分離したグループ用セッション
- **保留メッセージ** = メンション前のグループ発言の文脈

## 関連ページ

- [[concepts/groups]] / [[channels/whatsapp]] / [[sources/channels/groups]]
- [[sources/channels/channels]]
