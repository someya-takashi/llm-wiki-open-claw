---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/platforms/macos
source_path: raw/docs/platforms/macos.md
doc_section: platforms
title: "macOS アプリ"
ingested: 2026-06-14
tags: [platform, macos, menu-bar, companion-app, node, launchd]
related:
  - "[[components/node]]"
  - "[[components/control-ui]]"
  - "[[sources/platforms/platforms]]"
---

# macOS アプリ（解説）

> 原典: `raw/docs/platforms/macos.md` ・ https://docs.openclaw.ai/ja-JP/platforms/macos

## 一言まとめ

macOS アプリは OpenClaw の**メニューバー用コンパニオン**。権限管理・ローカル Gateway への接続/管理（launchd or 手動）・macOS 機能を **Node** としてエージェントに公開する。

## 位置づけ

[[components/node]] の macOS 実装（Mac Node モード）かつ [[components/control-ui]]・[[components/webchat]] のホスト。[[sources/platforms/platforms]] の OS 別ページ。

## 仕組み・ふるまい

- メニューバーから Gateway を起動/管理（LaunchAgent `ai.openclaw.gateway`）。camera/screen/location 等を Node コマンドとして公開（[[sources/nodes/nodes]]）。リモートモードでは SSH トンネルで `localhost` 接続（[[concepts/remote-access]]）。
- Talk/Voice Wake・Peekaboo ブリッジ・サイレント承認ペアリングなど macOS 固有機能。

## 設定・使い方の要点

- Settings でカメラ/exec 承認/Talk を制御。Gateway インストールは `openclaw gateway install`（launchd）。

## 用語と略称

- **メニューバーアプリ** = macOS 上部バー常駐の補助アプリ
- **launchd / LaunchAgent** = macOS のサービス常駐機構
- **Mac Node モード** = Mac を Node として Gateway に接続
- **Peekaboo** = macOS 自動化ブリッジ

## 関連ページ

- [[components/node]] / [[components/control-ui]] / [[sources/platforms/platforms]]
- [[sources/nodes/nodes]] / [[concepts/remote-access]] / [[concepts/voice]]
