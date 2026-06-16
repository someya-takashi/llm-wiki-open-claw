---
type: concept
aliases: [ACP, Agent Client Protocol, ACP エージェント, acpx]
tags: [acp, agent-client-protocol, harness, external-agent, runtime, binding]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/tasks]]"
components:
  - "[[components/plugin-system]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/tools/acp-agents]]"
  - "[[sources/tools/acp-agents-setup]]"
updated: 2026-06-14
---

# ACP（Agent Client Protocol）

ACP（Agent Client Protocol, 外部エージェントハーネスを標準プロトコルで制御する仕組み）は、OpenClaw が **外部のコーディングハーネス**——Pi・Claude Code・Cursor・Copilot・Droid・OpenCode・Gemini CLI ほか（総称 acpx）——を丸ごと走らせる手段。OpenClaw はチャネル・セッション・配信を担い、実際の作業はそのハーネスが所有する。

## なぜ重要か

「Claude Code のような本格的なコーディングエージェントを、メッセージング経由で動かしたい」とき、OpenClaw 自前のループ（PI）に作り込むより、**その専用ハーネスをそのまま使う**方が、ネイティブのスレッド再開・ツール継続・権限モデルを活かせる。ACP はそのための標準インターフェースで、OpenClaw を「多様なエージェントランタイムのフロントエンド」にする。

## エージェントランタイムの中での位置

[[concepts/agent-runtimes]] が整理する「ターンを実行するバックエンド」の選択肢：

| 層 | 何が実行するか |
|---|---|
| PI（既定） | OpenClaw 組み込みハーネス |
| Codex | OpenAI Codex app-server（[[sources/plugins/codex-harness]]） |
| CLI バックエンド | ローカル AI CLI をテキストで（[[sources/gateway/cli-backends]]） |
| **ACP** | **外部ハーネスを完全な ACP セッションで** |

CLI バックエンドが「テキスト専用の軽いフォールバック」なら、ACP は「ネイティブ再開/ツール継続を持つ重い完全ハーネス」。

## 仕組み（要点）

- **バインド済みセッション**：会話/スレッドを ACP セッションにバインドし、以降の通常テキストをそのハーネスへルーティング（Gateway 管理コマンドはローカル）。`/acp` 系コマンドで制御。
- **タスク追跡**：各 ACP セッション生成は [[concepts/tasks]] のバックグラウンドタスク。
- **MCP ブリッジ**：OpenClaw のツールを MCP でハーネスに公開。権限は `permissionMode`/`nonInteractivePermissions`。設定は [[sources/tools/acp-agents-setup]]。
- 実体は [[components/plugin-system]] の ACP バックエンド Plugin、仲介は [[components/gateway]]。

## 既存 wiki とのつながり

ACP は [[concepts/multi-agent]] の委任とも関わるが、OpenClaw 内の軽い委任実行（[[sources/tools/subagents]]）とは別物——ACP は外部ハーネスの本格セッション。実行中の誘導は [[sources/tools/steer]]、状態は `openclaw tasks`。非対話権限と MCP 公開範囲は [[concepts/security]]・[[concepts/exec]] の承認設計に従う。

## 代表ソース

- [[sources/tools/acp-agents]] — ACP の概念・ランブック・バインディング
- [[sources/tools/acp-agents-setup]] — acpx ハーネス設定・MCP ブリッジ・権限

## 関連ページ

- [[concepts/agent-runtimes]] / [[concepts/multi-agent]] / [[concepts/tasks]]
- [[components/plugin-system]] / [[components/gateway]] / [[concepts/security]]
