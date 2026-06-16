---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/network
source_path: raw/docs/network.md
doc_section: root
title: "ネットワーク"
ingested: 2026-06-14
tags: [network, hub, loopback, pairing, discovery, security]
related:
  - "[[concepts/discovery]]"
  - "[[concepts/pairing]]"
  - "[[components/gateway]]"
---

# ネットワーク（解説）

> 原典: `raw/docs/network.md` ・ https://docs.openclaw.ai/ja-JP/network
>
> ℹ️ docs の**ルート直下のハブページ**（localhost / LAN / tailnet での接続・ペアリング・保護の入口リンク集）。

## 一言まとめ

OpenClaw が「デバイスをどう**接続・ペアリング・保護**するか」の中核ドキュメントを束ねるハブ。中核モデルは「すべては Gateway を通る／まずループバック／ホストごと 1 Gateway」。

## 位置づけ

[[components/gateway]] を中心としたネットワーク像の入口。検出は [[concepts/discovery]]、信頼確立は [[concepts/pairing]]、安全側の既定は [[concepts/security]]・[[concepts/authentication]]。

## 中核モデル

- **まずループバック**：Gateway WS は既定 `ws://127.0.0.1:18789`。非ループバックバインドには有効な Gateway 認証経路（トークン/パスワード、または正しく構成した非ループバック `trusted-proxy`）が必須。
- **ホストごと 1 Gateway** 推奨。分離するなら別プロファイル・別ポートで複数起動（→ [[sources/gateway/multiple-gateways]]）。
- **Canvas ホスト**は Gateway と同ポートで提供（`/__openclaw__/canvas/`・`/__openclaw__/a2ui/`）。ループバックを越える場合は Gateway 認証で保護。
- **リモートアクセス**は通常 SSH トンネルか Tailscale VPN。

## このハブが指す先（wiki 内の対応ページ）

- 接続・契約：[[concepts/architecture]] / [[sources/gateway/protocol]] / [[sources/gateway/gateway]]
- ペアリング・ID：[[concepts/pairing]] / [[sources/gateway/pairing]]
- 検出・トランスポート：[[concepts/discovery]] / [[sources/gateway/discovery]] / [[sources/gateway/bonjour]]
- ノード：[[components/node]] / [[sources/gateway/bridge-protocol]]（レガシー）
- セキュリティ：[[concepts/security]] / [[sources/gateway/configuration]]

## 用語と略称

- **ループバック（loopback）** = `127.0.0.1`（同一ホスト内）
- **tailnet** = Tailscale の仮想プライベートネットワーク
- **trusted-proxy** = 前段の信頼済みリバースプロキシに認証を委譲するモード
- **Canvas / A2UI** = Gateway が提供する UI サーフェス

## 関連ページ

- [[concepts/discovery]] / [[concepts/pairing]] / [[concepts/security]]
- [[components/gateway]] / [[components/node]]
