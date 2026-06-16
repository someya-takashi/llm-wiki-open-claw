---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/sdk-channel-plugins
source_path: raw/docs/plugins/sdk-channel-plugins.md
doc_section: plugins
title: "チャネル Plugin の構築"
ingested: 2026-06-14
tags: [plugin, sdk, channel, messaging, pairing, mention-policy]
related:
  - "[[components/plugin-system]]"
  - "[[sources/plugins/building-plugins]]"
  - "[[concepts/messages]]"
---

# チャネル Plugin の構築（解説）

> 原典: `raw/docs/plugins/sdk-channel-plugins.md` ・ https://docs.openclaw.ai/ja-JP/plugins/sdk-channel-plugins

## 一言まとめ

OpenClaw を**メッセージングプラットフォームに接続するチャネル Plugin** の作り方。DM セキュリティ・ペアリング・返信スレッド化・アウトバウンドメッセージングを備えた動くチャネルを `api.registerChannel(...)` で実装する。

## 位置づけ

[[components/plugin-system]] の `registerChannel` を使う開発ガイド（基礎は [[sources/plugins/building-plugins]]）。完成物は他チャネルと同じ [[concepts/messages]] パイプライン・[[concepts/pairing]] に乗る（実例 [[channels/zalouser]]）。チャネル概念の横断ハブ `channel-routing` は今後。

## 仕組み・ふるまい

- **チャネル Plugin の仕組み**：受信メッセージを正規化して Gateway に渡し、Gateway の返信をプラットフォームへ送る。承認とチャネル機能（capabilities）・**インバウンドメンションポリシー**（グループでのメンション要求）を宣言。
- **ウォークスルー**：受信ハンドラ・アウトバウンド送信・返信スレッド化・DM ペアリングを順に実装。`channelConfigs.<id>.preferOver` で別 Plugin の同名チャネルを置換可能。

## 設定・使い方の要点

- ファイル構造（マニフェスト＋チャネル登録モジュール）。高度なトピック：スレッド化オプション、複数アカウント、メディア。チャネル設定は `channels.<id>` 配下に出る（`plugins.entries.*` ではない場合がある）。

## 注意点・落とし穴

- DM セキュリティとペアリングは必須要件として組み込む（[[concepts/pairing]]）。メンションポリシーを誤ると未承認送信者に応答してしまう（[[concepts/messages]] のメンションゲート）。

## 用語と略称

- **チャネル（channel）** = メッセージングプラットフォーム統合
- **`registerChannel`** = チャネルを登録する Plugin API
- **インバウンドメンションポリシー** = グループで応答するためのメンション要求設定
- **DM ペアリング** = 新規ダイレクト送信者の承認（[[concepts/pairing]]）

## 関連ページ

- [[components/plugin-system]] / [[sources/plugins/building-plugins]]
- [[concepts/messages]] / [[concepts/pairing]] / [[channels/zalouser]]
