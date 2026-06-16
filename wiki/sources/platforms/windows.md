---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/platforms/windows
source_path: raw/docs/platforms/windows.md
doc_section: platforms
title: "Windows"
ingested: 2026-06-14
tags: [platform, windows, wsl2, gateway, scheduled-task]
related:
  - "[[components/gateway]]"
  - "[[sources/platforms/platforms]]"
---

# Windows（解説）

> 原典: `raw/docs/platforms/windows.md` ・ https://docs.openclaw.ai/ja-JP/platforms/windows

## 一言まとめ

OpenClaw は**ネイティブ Windows と WSL2 の両方**をサポート。WSL2 がより安定で推奨（CLI/Gateway/ツールが Linux 内で完全互換）。ネイティブ Windows もコア CLI/Gateway に対応（いくつか注意点あり）。ネイティブコンパニオンアプリは計画中。

## 位置づけ

[[components/gateway]] を Windows で動かす入口（[[sources/platforms/platforms]]）。Gateway サービスはスケジュールタスク。

## 仕組み・ふるまい

- **WSL2（推奨）**：Linux 内で systemd ユーザーサービスとして Gateway を実行（完全互換）。
- **ネイティブ Windows**：スケジュールタスク（`OpenClaw Gateway`）。タスク作成拒否時は Startup フォルダーのログイン項目にフォールバック。

## 設定・使い方の要点

- `openclaw gateway install`。WSL2 では Linux 手順に従う。Bun は Gateway 非推奨。

## 注意点・落とし穴

- ⚠️ ネイティブ Windows はノードホストの allowlist で `cmd.exe /c` 等のシェルラッパーに承認が必要（[[sources/nodes/nodes]]）。完全な体験は WSL2 推奨。

## 用語と略称

- **WSL2** = Windows Subsystem for Linux 2（Linux 互換層）
- **スケジュールタスク** = Windows のサービス常駐機構
- **systemd ユーザーサービス** = WSL2 内のサービス常駐

## 関連ページ

- [[components/gateway]] / [[sources/platforms/platforms]] / [[sources/platforms/linux]]
- [[sources/nodes/nodes]]
