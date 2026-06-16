---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/discovery
source_path: raw/docs/gateway/discovery.md
doc_section: gateway
title: "検出とトランスポート"
ingested: 2026-06-14
tags: [discovery, transport, bonjour, dns-sd, tailnet, ssh]
related:
  - "[[concepts/discovery]]"
  - "[[components/gateway]]"
  - "[[concepts/pairing]]"
---

# 検出とトランスポート（解説）

> 原典: `raw/docs/gateway/discovery.md` ・ https://docs.openclaw.ai/ja-JP/gateway/discovery

## 一言まとめ

クライアント（Mac アプリ・iOS/Android Node）が **Gateway の場所をどう見つけ、どの経路でつなぐか**。設計目標は「検出・広告はすべて Gateway（`openclaw gateway`）に集約、クライアントは利用側に徹する」。

## 位置づけ

[[concepts/discovery]] の本体。見つけた後の信頼確立は [[concepts/pairing]]、接続後のワイヤー契約は [[sources/gateway/protocol]]。[[components/gateway]] が検出ビーコンの広告主かつ WS エンドポイントのホスト。

## 仕組み・ふるまい

OpenClaw が解く 2 つの似て非なる問題：①**オペレーターのリモート制御**（Mac アプリが別ホストの Gateway を操作）／②**Node ペアリング**（モバイルが Gateway を発見し安全にペア）。

トランスポートは 2 系統を併存：
- **Direct WS**：LAN/tailnet 向けの Gateway WS エンドポイント（SSH 不要）。同一ネット・tailnet 内で最良の UX、プロトコル面が小さく監査しやすい。
- **SSH（フォールバック）**：`127.0.0.1:18789` を SSH 転送。SSH が通る場所ならどこでも動き、mDNS の問題を回避、新規ポート不要。

**検出入力（3 つ）**：
1. **Bonjour / DNS-SD**：multicast はベストエフォートで LAN 内のみ。広域 unicast DNS-SD ドメイン経由で同じビーコンを参照すればネットワーク横断も可能。詳細は [[sources/gateway/bonjour]]。
2. **Tailnet**：別ネット間（例 London/Vienna）では Bonjour が効かないので Tailscale MagicDNS 名（推奨）/安定 tailnet IP を direct ターゲットに。macOS アプリは生 IP より MagicDNS 名を優先（IP 変動に強い）。
3. **手動 / SSH**：direct が無い/無効なら loopback ポートを SSH 転送（→ [[sources/gateway/remote]] / [[concepts/remote-access]]）。

## 設定・使い方の要点

- **トランスポート選択ポリシー（クライアント）**：①到達可能なペアリング済み direct → ②検出された Gateway をワンタップ採用 → ③設定済み tailnet direct → ④SSH フォールバック。
- direct トランスポートでも Gateway が認証（トークン/キーペア）・スコープ/ACL・レート制限を強制（生プロキシではない）。

## 注意点・落とし穴

- **検出ヒントは tailnet/public 経路のトランスポートセキュリティを緩めない**：モバイル Node は初回経路に `wss://` か Tailscale Serve/Funnel を要求。生 tailnet IP は「ルーティングヒント」であって平文 `ws://` 許可ではない（プライベート LAN の `ws://` は引き続き可）。

## 用語と略称

- **検出（discovery）** = Gateway の所在を知る手段
- **トランスポート（transport）** = 実際の接続経路（Direct WS / SSH）
- **DNS-SD** = DNS Service Discovery（DNS でサービスを公告/参照）
- **tailnet** = Tailscale が作る仮想プライベートネットワーク
- **MagicDNS** = Tailscale のホスト名解決（IP 変動を吸収）

## 関連ページ

- [[concepts/discovery]] — 対応する概念ページ
- [[sources/gateway/bonjour]] / [[sources/gateway/protocol]] / [[sources/gateway/pairing]]
- [[sources/network]] / [[sources/gateway/bridge-protocol]] / [[components/gateway]]
