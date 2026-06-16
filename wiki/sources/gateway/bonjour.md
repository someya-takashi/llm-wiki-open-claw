---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/bonjour
source_path: raw/docs/gateway/bonjour.md
doc_section: gateway
title: "Bonjour"
ingested: 2026-06-14
tags: [bonjour, mdns, dns-sd, discovery, beacon, plugin]
related:
  - "[[concepts/discovery]]"
  - "[[sources/gateway/discovery]]"
  - "[[components/gateway]]"
---

# Bonjour（解説）

> 原典: `raw/docs/gateway/bonjour.md` ・ https://docs.openclaw.ai/ja-JP/gateway/bonjour

## 一言まとめ

Gateway が LAN/広域 DNS-SD 上に **WS エンドポイントを広告するためのビーコン**（同梱 `bonjour` Plugin）。サービスタイプ `_openclaw-gw._tcp` を出し、クライアントは「Gateway を選択」リストに使う。

## 位置づけ

[[sources/gateway/discovery]] の検出入力①の詳細。[[concepts/discovery]] の構成要素で、[[components/gateway]] が広告主。mDNS は LAN 限定、広域は unicast DNS-SD ドメイン。

## 仕組み・ふるまい

- **サービスタイプ**：`_openclaw-gw._tcp`（Gateway トランスポートビーコン）。
- **TXT キー（非シークレット）**：`role=gateway`・`transport=gateway`・`displayName`・`lanHost=<host>.local`・`gatewayPort=18789`（WS+HTTP）・`gatewayTls=1`/`gatewayTlsSha256=<sha256>`（TLS 有効時のみ）・`canvasPort`・`tailnetDns`・`sshPort`/`cliPath`（mDNS フルモードのみ。広域 DNS-SD では省略され SSH は既定 `22`）。

## 設定・使い方の要点

- 有効化：`openclaw plugins enable bonjour`（LAN multicast 広告 ON）。
- **macOS Gateway は空設定起動で自動有効**、Linux/Windows/コンテナは明示有効化が必要。検出されたコンテナ内では自動無効化。
- 環境変数：`OPENCLAW_DISABLE_BONJOUR=1`（無効）/`0`（強制有効、host/macvlan 等のみ）、`OPENCLAW_SSH_PORT`・`OPENCLAW_TAILNET_DNS`・`OPENCLAW_CLI_PATH` で広告値を上書き。バインドは `gateway.bind`。

## 注意点・落とし穴

- ⚠️ **TXT レコードは認証されない**：クライアントは TXT を UX ヒントとしてのみ扱う。ルーティングは TXT の `lanHost`/`gatewayPort` より**解決済みサービスエンドポイント（SRV + A/AAAA）を優先**。
- ⚠️ **TLS pinning**：広告された `gatewayTlsSha256` が保存済み pin を上書きすることを絶対に許さない。セキュア経路の初回 pin 保存前に帯域外の「このフィンガープリントを信頼する」確認を要求。

## 用語と略称

- **Bonjour / mDNS** = Apple のゼロ設定ネットワーキング（multicast DNS）
- **DNS-SD** = DNS Service Discovery
- **TXT レコード** = サービスのメタデータを載せる DNS レコード
- **SRV / A / AAAA** = サービス所在・IPv4・IPv6 を解決する DNS レコード
- **TLS pinning** = 特定証明書フィンガープリントだけを信頼する固定

## 関連ページ

- [[concepts/discovery]] / [[sources/gateway/discovery]]
- [[components/gateway]] / [[sources/network]]
