---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/platforms/ios
source_path: raw/docs/platforms/ios.md
doc_section: platforms
title: "iOS アプリ"
ingested: 2026-06-14
tags: [platform, ios, mobile-node, companion-app, preview]
related:
  - "[[components/node]]"
  - "[[sources/platforms/platforms]]"
  - "[[concepts/pairing]]"
---

# iOS アプリ（解説）

> 原典: `raw/docs/platforms/ios.md` ・ https://docs.openclaw.ai/ja-JP/platforms/ios

## 一言まとめ

iOS アプリは OpenClaw の**モバイルノード**で、iPhone/iPad の機能（カメラ・画面・位置・音声）を Gateway 経由でエージェントに公開する。現在は**内部プレビュー**（一般公開前）。

## 位置づけ

[[components/node]] の iOS 実装。[[sources/platforms/platforms]] の OS 別ページで、接続は [[concepts/pairing]]（デバイスペアリング）・[[concepts/discovery]]（Bonjour/tailnet）。

## 仕組み・ふるまい

- OpenClaw ノードとして Gateway WS に接続し、`canvas.*`/`camera.*`/`screen.*`/`location.*`/`talk.*` を公開（[[sources/nodes/nodes]]）。Voice Wake・Talk 対応。
- tailnet/public 経路は `wss://` か Tailscale Serve を要求（[[concepts/discovery]]）。

## 設定・使い方の要点

- Gateway を Bonjour/tailnet で発見しデバイスペアリングを承認（`openclaw devices approve`）。

## 注意点・落とし穴

- 内部プレビュー段階。モバイルノードは tailnet/public でセキュアトランスポート必須。

## 用語と略称

- **モバイルノード** = スマホ/タブレットを Node として接続
- **デバイスペアリング** = ノードの承認とトークン発行（[[concepts/pairing]]）
- **Voice Wake** = ウェイクワード起動（[[concepts/voice]]）

## 関連ページ

- [[components/node]] / [[sources/platforms/platforms]] / [[concepts/pairing]] / [[concepts/discovery]]
- [[sources/nodes/nodes]] / [[concepts/voice]]
