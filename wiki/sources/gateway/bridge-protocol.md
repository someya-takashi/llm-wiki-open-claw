---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/bridge-protocol
source_path: raw/docs/gateway/bridge-protocol.md
doc_section: gateway
title: "ブリッジプロトコル（レガシー）"
ingested: 2026-06-14
tags: [bridge, legacy, removed, transport, history]
related:
  - "[[concepts/discovery]]"
  - "[[sources/gateway/protocol]]"
---

# ブリッジプロトコル（レガシー）（解説）

> 原典: `raw/docs/gateway/bridge-protocol.md` ・ https://docs.openclaw.ai/ja-JP/gateway/bridge-protocol
>
> ℹ️ **削除済みの歴史的トランスポート**。現行ビルドには含まれず、検出でも広告されない。新規構築では無関係——[[sources/gateway/protocol]]（Gateway WS）を使う。

## 一言まとめ

ブリッジプロトコルは旧来の Node トランスポート（TCP ベース）。現在は **Gateway WS プロトコルに置き換えられて削除済み**で、歴史的経緯と移行のために残る記述。

## 位置づけ

[[concepts/discovery]] が参照する「もう使わない経路」。現行のノード接続は Direct WS か SSH フォールバック（[[sources/gateway/discovery]]）、ワイヤー契約は [[sources/gateway/protocol]]。

## 仕組み・ふるまい（歴史）

- かつてノードは独自の TCP ブリッジで Gateway につないでいた。
- 現在は **Bonjour 検出でも広告されず**、ビルドにも含まれない。メンバーシップ判断・トランスポートはすべて Gateway WS に集約された。

## 注意点・落とし穴

- 古いドキュメント/設定がブリッジ前提なら **WS（`gateway.bind` + ペアリング）へ移行**する。新規にブリッジを有効化する手段は無い。

## 用語と略称

- **ブリッジ（bridge）** = 旧 Node トランスポート（TCP）
- **WS** = WebSocket（現行トランスポート）

## 関連ページ

- [[concepts/discovery]] / [[sources/gateway/discovery]]
- [[sources/gateway/protocol]] / [[sources/gateway/pairing]]
