---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/channel-routing
source_path: raw/docs/channels/channel-routing.md
doc_section: channels
title: "チャネルとルーティング"
ingested: 2026-06-14
tags: [channel-routing, routing, session-key, reply-routing, broadcast]
related:
  - "[[concepts/channel-routing]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/session]]"
---

# チャネルとルーティング（解説）

> 原典: `raw/docs/channels/channel-routing.md` ・ https://docs.openclaw.ai/ja-JP/channels/channel-routing

## 一言まとめ

OpenClaw は返信を**メッセージの送信元チャネルへ戻す**ように決定的にルーティングする。モデルはチャネルを選ばない——ルーティングはホスト設定で制御される。

## 位置づけ

[[concepts/channel-routing]] の中核ソース。どのチャネルのどの会話が、どのエージェント/セッション（[[concepts/multi-agent]] のバインディング）に向かうかを定める。返信先の付け替えは [[concepts/channel-docking]]。

## 仕組み・ふるまい

- **決定的ルーティング**：返信は常に送信元チャネルへ。送信先プレフィックス（`telegram:123` 等）で宛先を表す。
- **セッションキーの形状**：`agent:<agentId>:<channel>:<peer>` 系（[[concepts/session]]）。**メイン DM ルートのピン留め**で `main` セッションを固定。
- **ルーティングルール**：受信メッセージからエージェントを選ぶ（バインディング）。**ガードされた受信記録**で不正な受信を弾く。
- **ブロードキャストグループ**：1 グループに複数エージェントを走らせる（[[sources/channels/broadcast-groups]]）。

## 設定・使い方の要点

- 設定概要・セッションストレージ・WebChat の動作（返信は常に WebChat に戻る）。返信コンテキスト（送信者ラベル等）。

## 注意点・落とし穴

- モデルは宛先を選べない（ルーティングはホストが決める）。これが [[concepts/security]] 上重要——エージェントが勝手に別チャネルへ送れない。

## 用語と略称

- **ルーティング（routing）** = どの会話をどのエージェント/宛先に振り分けるか
- **送信先プレフィックス** = `<channel>:<peer>` の宛先表記
- **セッションキー** = `agent:<id>:<channel>:<peer>` 形式の識別子
- **バインディング** = チャネル/会話とエージェントの結びつけ

## 関連ページ

- [[concepts/channel-routing]] — 対応する概念ページ
- [[concepts/multi-agent]] / [[concepts/session]] / [[concepts/channel-docking]]
- [[sources/channels/groups]] / [[sources/channels/broadcast-groups]]
