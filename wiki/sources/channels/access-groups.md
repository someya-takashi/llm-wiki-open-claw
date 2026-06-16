---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/access-groups
source_path: raw/docs/channels/access-groups.md
doc_section: channels
title: "アクセスグループ"
ingested: 2026-06-14
tags: [access-groups, allowlist, sender-group, authorization, channels]
related:
  - "[[concepts/groups]]"
  - "[[concepts/security]]"
  - "[[concepts/channel-routing]]"
---

# アクセスグループ（解説）

> 原典: `raw/docs/channels/access-groups.md` ・ https://docs.openclaw.ai/ja-JP/channels/access-groups

## 一言まとめ

アクセスグループは、一度定義してチャネル許可リストから `accessGroup:<name>` で参照する**名前付きの送信者リスト**。同じ人たちを複数チャネルで許可したいときに、信頼集合を 1 箇所で管理できる。

## 位置づけ

[[concepts/groups]] の認可（誰の発言を受け付けるか）面で、[[concepts/security]] の許可リストを再利用可能にする。送信者認可（誰がエージェントに話せるか）であって、オペレータースコープ（[[sources/gateway/operator-scopes]]、認証後に何ができるか）とは別レイヤー。

## 仕組み・ふるまい

- **静的メッセージ送信者グループ**：`accessGroups.<name>: [送信者...]` を定義。
- **許可リストから参照**：チャネルの `allowFrom` 等に `accessGroup:<name>` を書くと、その集合が展開される。
- ⚠️ **アクセスグループ単体ではアクセスを付与しない**——許可リストフィールドが参照して初めて意味を持つ。

## 設定・使い方の要点

- サポートされるメッセージチャネルパス（どの `allowFrom` で使えるか）。Plugin 診断・Discord チャネルオーディエンス（チャネルメンバーを集合に）。

## 注意点・落とし穴

- セキュリティメモ：信頼集合を広げすぎない。グループは定義であって付与ではないので、参照漏れに注意（トラブルシュートあり）。

## 用語と略称

- **アクセスグループ** = `accessGroup:<name>` で参照する名前付き送信者集合
- **許可リスト（allowlist）** = 受け付ける送信者の指定（`allowFrom` 等）
- **送信者認可** = 誰がエージェントに話せるか（≠ 認証後の権限）

## 関連ページ

- [[concepts/groups]] / [[concepts/security]] / [[concepts/channel-routing]]
- [[concepts/pairing]] / [[sources/gateway/operator-scopes]]
