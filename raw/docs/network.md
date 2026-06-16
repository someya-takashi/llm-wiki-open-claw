---
title: "ネットワーク"
source: "https://docs.openclaw.ai/ja-JP/network"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
このハブは、OpenClaw が localhost、LAN、tailnet 全体でデバイスを接続、ペアリング、保護する方法についての中核ドキュメントへのリンク集です。

## 中核モデル

ほとんどの操作は Gateway (`openclaw gateway`) を通じて流れます。これはチャネル接続と WebSocket 制御プレーンを所有する、単一の長時間実行プロセスです。

- **まずループバック**: Gateway WS のデフォルトは `ws://127.0.0.1:18789` です。 非ループバックのバインドには、有効な gateway 認証パスが必要です。共有シークレットの トークン/パスワード認証、または正しく構成された非ループバックの `trusted-proxy` デプロイです。
- **ホストごとに 1 つの Gateway** を推奨します。分離する場合は、分離されたプロファイルとポートで複数の gateway を実行してください（ [複数の Gateway](https://docs.openclaw.ai/ja-JP/gateway/multiple-gateways) ）。
- **Canvas ホスト** は Gateway と同じポートで提供されます（ `/__openclaw__/canvas/` 、 `/__openclaw__/a2ui/` ）。ループバックを超えてバインドされる場合は Gateway 認証で保護されます。
- **リモートアクセス** は通常、SSH トンネルまたは Tailscale VPN です（ [リモートアクセス](https://docs.openclaw.ai/ja-JP/gateway/remote) ）。

主な参照先:

- [Gateway アーキテクチャ](https://docs.openclaw.ai/ja-JP/concepts/architecture)
- [Gateway プロトコル](https://docs.openclaw.ai/ja-JP/gateway/protocol)
- [Gateway ランブック](https://docs.openclaw.ai/ja-JP/gateway)
- [Web サーフェス + バインドモード](https://docs.openclaw.ai/ja-JP/web)

## ペアリング + アイデンティティ

- [ペアリング概要（DM + ノード）](https://docs.openclaw.ai/ja-JP/channels/pairing)
- [Gateway 所有のノードペアリング](https://docs.openclaw.ai/ja-JP/gateway/pairing)
- [デバイス CLI（ペアリング + トークンローテーション）](https://docs.openclaw.ai/ja-JP/cli/devices)
- [ペアリング CLI（DM 承認）](https://docs.openclaw.ai/ja-JP/cli/pairing)

ローカル信頼:

- 直接の local loopback 接続は、同一ホストでの UX を滑らかに保つため、 ペアリングを自動承認できます。
- OpenClaw には、信頼された共有シークレットのヘルパーフロー向けに、狭い backend/container-local の自己接続パスもあります。
- 同一ホストの tailnet バインドを含む tailnet および LAN クライアントには、 それでも明示的なペアリング承認が必要です。

## 検出 + トランスポート

- [検出とトランスポート](https://docs.openclaw.ai/ja-JP/gateway/discovery)
- [Bonjour / mDNS](https://docs.openclaw.ai/ja-JP/gateway/bonjour)
- [リモートアクセス（SSH）](https://docs.openclaw.ai/ja-JP/gateway/remote)
- [Tailscale](https://docs.openclaw.ai/ja-JP/gateway/tailscale)

## ノード + トランスポート

- [ノード概要](https://docs.openclaw.ai/ja-JP/nodes)
- [Bridge プロトコル（レガシーノード、履歴）](https://docs.openclaw.ai/ja-JP/gateway/bridge-protocol)
- [ノードランブック: iOS](https://docs.openclaw.ai/ja-JP/platforms/ios)
- [ノードランブック: Android](https://docs.openclaw.ai/ja-JP/platforms/android)

## セキュリティ

- [セキュリティ概要](https://docs.openclaw.ai/ja-JP/gateway/security)
- [Gateway 構成リファレンス](https://docs.openclaw.ai/ja-JP/gateway/configuration)
- [トラブルシューティング](https://docs.openclaw.ai/ja-JP/gateway/troubleshooting)
- [Doctor](https://docs.openclaw.ai/ja-JP/gateway/doctor)

## 関連

- [Gateway ランブック](https://docs.openclaw.ai/ja-JP/gateway)
- [リモートアクセス](https://docs.openclaw.ai/ja-JP/gateway/remote)