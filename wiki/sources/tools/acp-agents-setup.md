---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/acp-agents-setup
source_path: raw/docs/tools/acp-agents-setup.md
doc_section: tools
title: "ACP エージェント — セットアップ"
ingested: 2026-06-14
tags: [acp, setup, acpx, mcp-bridge, permissions, plugin]
related:
  - "[[concepts/acp]]"
  - "[[sources/tools/acp-agents]]"
  - "[[components/plugin-system]]"
---

# ACP エージェント — セットアップ（解説）

> 原典: `raw/docs/tools/acp-agents-setup.md` ・ https://docs.openclaw.ai/ja-JP/tools/acp-agents-setup

## 一言まとめ

ACP の**設定編**：acpx ハーネスの設定・MCP ブリッジ向け Plugin セットアップ・権限設定。概念とランブックは [[sources/tools/acp-agents]]。

## 位置づけ

[[concepts/acp]] の運用詳細。ACP ハーネスは [[components/plugin-system]] の ACP バックエンド Plugin として登録され、OpenClaw ツールを MCP（Model Context Protocol）でハーネスに橋渡しする。

## 仕組み・ふるまい

- **acpx ハーネスサポート**：対応ハーネスの一覧と必須設定。**Plugin セットアップ**：acpx コマンド/バージョン設定・自動依存インストール・ランタイムタイムアウト・ヘルスプローブエージェント。
- **MCP ブリッジ**：Plugin ツール MCP ブリッジ／OpenClaw ツール MCP ブリッジで、OpenClaw のツールを ACP ハーネスから使えるようにする。
- **権限**：`permissionMode`・`nonInteractivePermissions`（非対話実行の承認方針）。

## 設定・使い方の要点

- 各 acpx ハーネスのコマンド/バージョンを宣言し、必要なら自動依存インストール。OpenClaw の exec ポリシー（[[concepts/exec]]）と権限モードを対応づける。

## 注意点・落とし穴

- 非対話権限の設計は重要（自動承認はリスク）。MCP ブリッジ経由で OpenClaw ツールを公開する範囲を絞る（[[concepts/security]]）。

## 用語と略称

- **acpx** = 対応 ACP ハーネスの総称
- **MCP ブリッジ** = OpenClaw ツールを MCP でハーネスに公開する仕組み
- **permissionMode / nonInteractivePermissions** = 権限の対話/非対話モード
- **ヘルスプローブ** = ハーネスの稼働確認用エージェント

## 関連ページ

- [[concepts/acp]] / [[sources/tools/acp-agents]] / [[components/plugin-system]]
- [[concepts/exec]] / [[concepts/security]]
