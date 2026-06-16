---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/acp-agents
source_path: raw/docs/tools/acp-agents.md
doc_section: tools
title: "ACP エージェント"
ingested: 2026-06-14
tags: [acp, agent-client-protocol, harness, claude-code, binding, runtime]
related:
  - "[[concepts/acp]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/tasks]]"
---

# ACP エージェント（解説）

> 原典: `raw/docs/tools/acp-agents.md` ・ https://docs.openclaw.ai/ja-JP/tools/acp-agents

## 一言まとめ

Agent Client Protocol（ACP）セッションにより、OpenClaw は **外部コーディングハーネス**（Pi・Claude Code・Cursor・Copilot・Droid・OpenCode・Gemini CLI ほか）を ACP バックエンド Plugin 経由で実行できる。各セッション生成は [[concepts/tasks]] のバックグラウンドタスクとして追跡される。

## 位置づけ

[[concepts/acp]] の中核ソース。[[concepts/agent-runtimes]] の「外部ハーネスを丸ごと走らせる」ランタイムで、OpenClaw はチャネル/セッション/配信を担い、実際のコーディングは ACP ハーネスが所有する。CLI バックエンド（[[sources/gateway/cli-backends]]）より重い「完全なハーネス」契約。

## 仕組み・ふるまい

- **バインド済みセッション**：会話/スレッドを ACP セッションにバインドし、以降の通常テキストをそのハーネスへルーティング（Gateway 管理コマンドはローカルに留まる）。`/acp` 系コマンドで制御。
- **永続的なチャンネルバインディング**：チャネルごと/エージェントごとのランタイム既定。
- **ACP vs サブエージェント**：ACP は外部ハーネスの本格セッション（ネイティブ再開/ツール継続）、サブエージェント（[[sources/tools/subagents]]）は OpenClaw 内の軽い委任実行。

## 設定・使い方の要点

- オペレーター向けランブック（生成/キャンセル/再開）。`/acp spawn|cancel|steer|close|sessions|status|set-mode|...`（[[concepts/slash-commands]]）。詳細なハーネス設定・権限・MCP ブリッジは [[sources/tools/acp-agents-setup]]。

## 注意点・落とし穴

- ACP セッションにバインド中は通常フォローアップがハーネスにルーティングされる。誘導は [[sources/tools/steer]]、状態は `openclaw tasks`（[[sources/automation/tasks]]）。

## 用語と略称

- **ACP** = Agent Client Protocol（外部エージェントハーネスを制御する標準）
- **ハーネス（harness）** = ターンを実行するバックエンド（Claude Code 等）
- **acpx** = 対応 ACP ハーネスの総称
- **バインド済みセッション** = 会話を特定 ACP ハーネスに束ねた状態

## 関連ページ

- [[concepts/acp]] — 対応する概念ページ
- [[sources/tools/acp-agents-setup]] / [[concepts/agent-runtimes]] / [[concepts/tasks]]
- [[sources/tools/subagents]] / [[sources/gateway/cli-backends]]
