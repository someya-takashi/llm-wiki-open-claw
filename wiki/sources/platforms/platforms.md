---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/platforms
source_path: raw/docs/platforms/platforms.md
doc_section: platforms
title: "プラットフォーム（OS 別）"
ingested: 2026-06-14
tags: [platforms, os, node, runtime, companion-app, service]
related:
  - "[[components/gateway]]"
  - "[[components/node]]"
  - "[[concepts/configuration]]"
---

# プラットフォーム（OS 別）（解説）

> 原典: `raw/docs/platforms/platforms.md` ・ https://docs.openclaw.ai/ja-JP/platforms
>
> ℹ️ `platforms/` セクションのランディング。OS ごとの実行・コンパニオンアプリの入口。

## 一言まとめ

OpenClaw コアは TypeScript で、**Node が推奨ランタイム**（Bun は Gateway 非推奨）。macOS（メニューバーアプリ）とモバイルノード（iOS/Android）にコンパニオンアプリがあり、Windows/Linux は Gateway が完全サポート（アプリは計画中）。

## 位置づけ

各 OS で [[components/gateway]] を動かす入口。モバイルアプリは [[components/node]]、macOS アプリは Node＋[[components/control-ui]] ホスト。VPS ホスティング（Fly/Hetzner/GCP/Azure）も束ねる。

## 仕組み・ふるまい

- **OS 別**：[[sources/platforms/macos]]（メニューバー）・[[sources/platforms/ios]]/[[sources/platforms/android]]（モバイルノード）・[[sources/platforms/windows]]（ネイティブ/WSL2）・[[sources/platforms/linux]]。
- **Gateway サービスインストール**：`openclaw onboard --install-daemon`/`gateway install`/`configure`/`doctor`。ターゲットは OS 依存——macOS=LaunchAgent（`ai.openclaw.gateway`）、Linux/WSL2=systemd ユーザーサービス、Windows=スケジュールタスク。

## 設定・使い方の要点

- VPS：Fly.io/Hetzner(Docker)/GCP/Azure/exe.dev。Windows は WSL2 推奨（完全互換）。

## 用語と略称

- **コンパニオンアプリ** = OS ネイティブの補助アプリ（メニューバー/モバイル）
- **WSL2** = Windows Subsystem for Linux 2
- **LaunchAgent / systemd / スケジュールタスク** = OS 別のサービス常駐機構
- **Node ランタイム** = OpenClaw の推奨実行環境（≠ OpenClaw の Node 概念）

## 関連ページ

- [[components/gateway]] / [[components/node]] / [[components/control-ui]]
- [[sources/platforms/macos]] / [[sources/platforms/ios]] / [[sources/platforms/android]] / [[sources/platforms/windows]] / [[sources/platforms/linux]]
