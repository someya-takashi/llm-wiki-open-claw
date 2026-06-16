---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/platforms/linux
source_path: raw/docs/platforms/linux.md
doc_section: platforms
title: "Linux アプリ"
ingested: 2026-06-14
tags: [platform, linux, gateway, systemd, node-runtime]
related:
  - "[[components/gateway]]"
  - "[[sources/platforms/platforms]]"
---

# Linux アプリ（解説）

> 原典: `raw/docs/platforms/linux.md` ・ https://docs.openclaw.ai/ja-JP/platforms/linux

## 一言まとめ

Gateway は Linux で**完全サポート**。**Node が推奨ランタイム**（Bun は WhatsApp/Telegram の不具合で非推奨）。ネイティブ Linux コンパニオンアプリは計画中。

## 位置づけ

[[components/gateway]] の主要な本番環境（VPS/サーバー、[[sources/platforms/platforms]]）。Gateway サービスは systemd ユーザーサービス。

## 仕組み・ふるまい

- `openclaw gateway install` で systemd ユーザーサービス（`openclaw-gateway.service`）。VPS（Fly/Hetzner/GCP/Azure）での常時稼働に向く。
- Linux Gateway でも macOS ノードが接続すれば macOS 専用 skill を `host=node` で実行可（[[sources/tools/skills]]）。

## 設定・使い方の要点

- Node ランタイムをインストールし `openclaw gateway install`。Bonjour は Linux では明示有効化（[[sources/gateway/bonjour]]）。

## 用語と略称

- **systemd ユーザーサービス** = Linux のサービス常駐機構
- **VPS** = Virtual Private Server（仮想専用サーバー）
- **Node ランタイム** = OpenClaw の推奨実行環境

## 関連ページ

- [[components/gateway]] / [[sources/platforms/platforms]] / [[sources/platforms/windows]]
- [[concepts/remote-access]] / [[sources/gateway/bonjour]]
