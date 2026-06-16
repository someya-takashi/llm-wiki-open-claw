---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/pairing
source_path: raw/docs/channels/pairing.md
doc_section: channels
title: "ペアリング（DM + ノード）"
ingested: 2026-06-14
tags: [pairing, dm-pairing, node-pairing, access-approval, allowlist]
related:
  - "[[concepts/pairing]]"
  - "[[concepts/channel-routing]]"
  - "[[components/node]]"
---

# ペアリング（DM + ノード）（解説）

> 原典: `raw/docs/channels/pairing.md` ・ https://docs.openclaw.ai/ja-JP/channels/pairing

## 一言まとめ

ペアリングは OpenClaw の**明示的なアクセス承認ステップ**で、2 か所で使われる：①**DM ペアリング**（ボットと会話できる人の承認）、②**Node ペアリング**（Gateway ネットワークに参加できるデバイス/Node の承認）。

## 位置づけ

[[concepts/pairing]] の統合的な入口。DM 側は [[concepts/channel-routing]] の送信者認可、Node 側は [[components/node]] のデバイス信頼（詳細は [[sources/gateway/pairing]]）。

## 仕組み・ふるまい

- **① DM ペアリング**：未知の DM 送信者を承認する（受信チャットアクセス）。送信者承認・再利用可能な送信者グループ（[[sources/channels/access-groups]]）。状態は保存される。
- **② Node デバイスペアリング**：iOS/Android/macOS/ヘッドレス Node を承認。**Telegram 経由ペアリング**（iOS 推奨）・`openclaw devices approve`・**信頼済み CIDR による自動承認**（[[sources/gateway/pairing]]）。状態保存あり。

## 設定・使い方の要点

- DM：チャネルの `dmPolicy: "pairing"`（多くのチャネル既定）。送信者を承認すると以降は許可リスト扱い。
- Node：`openclaw devices list|approve|reject`、QR/Telegram ペアリング。`gateway.nodes.pairing.autoApproveCidrs`。

## 注意点・落とし穴

- DM ペアリングと Node ペアリングは別物（前者は「誰が話せるか」、後者は「どのデバイスが繋げるか」）。両者は [[concepts/authentication]] の異なるレイヤー。

## 用語と略称

- **DM ペアリング** = ダイレクトメッセージ送信者の承認
- **Node ペアリング** = デバイス/Node の承認とトークン発行
- **CIDR** = ネットワーク範囲表記（自動承認の対象）
- **dmPolicy** = DM の受け入れ方針（pairing 等）

## 関連ページ

- [[concepts/pairing]] — 対応する概念ページ
- [[concepts/channel-routing]] / [[components/node]] / [[concepts/authentication]]
- [[sources/gateway/pairing]] / [[sources/channels/access-groups]]
