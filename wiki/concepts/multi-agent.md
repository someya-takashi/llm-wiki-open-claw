---
type: concept
aliases: [Multi-agent, マルチエージェント, multi-agent routing]
tags: [multi-agent, routing, bindings, isolation]
related:
  - "[[concepts/agent]]"
  - "[[concepts/session]]"
  - "[[concepts/delegate-architecture]]"
  - "[[concepts/parallel-specialist-lanes]]"
  - "[[concepts/sandboxing]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/multi-agent]]"
updated: 2026-06-14
---

# マルチエージェントルーティング

**マルチエージェント**は、1 つの [[components/gateway]] 内で**複数の分離されたエージェント**を並べて動かし、受信メッセージを**バインディング**で適切なエージェント（`agentId`）へ決定的にルーティングする仕組み。各エージェントは独自のワークスペース・状態ディレクトリ（`agentDir`）・セッション・認証を持つ。

## なぜ重要か

「1 台の Gateway を複数人・複数人格で共有しつつ、頭脳とデータを分離する」ための土台。[[concepts/agent]] の「1 プロセスの契約」を並立させ、`accountId`（チャネルアカウント）と `binding`（`(channel, accountId, peer)` → `agentId`）で**誰のメッセージがどの頭脳に届くか**を確定させる。これがプライバシー境界（[[concepts/session]] の DM 分離）とセキュリティ境界（エージェントごとの [[concepts/sandboxing]]・ツールポリシー）の前提になる。

## 押さえる点

- **ルーティングは決定的・最も具体的なものが優先**：peer → parentPeer → guildId+roles → guildId → teamId → accountId → チャネル → デフォルト。
- **エージェントごと境界**：`agents.list[].sandbox`・`tools.allow/deny` を Gateway レベルで強制（人格ファイルとは独立）。`tools.elevated` はグローバル。
- **`agentDir` を再利用しない**（認証/セッション衝突）。OAuth リフレッシュトークンはセカンダリへ複製されない。
- 応用：組織デプロイは [[concepts/delegate-architecture]]、並列ワーク設計は [[concepts/parallel-specialist-lanes]]。

設定例（Discord/Telegram/WhatsApp ボット、DM 分割、クロスエージェント検索）は [[sources/concepts/multi-agent]]。

## 関連

- [[concepts/agent]] / [[concepts/session]] / [[concepts/session-tool]]
- [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]]
- [[concepts/sandboxing]] / [[concepts/acp]] / [[concepts/channel-routing]]
- [[sources/tools/subagents]]（`sessions_spawn`） / [[sources/tools/multi-agent-sandbox-tools]] — エージェントごとの権限
- [[sources/prose]] — OpenProse（`.prose` で明示的並列のマルチエージェントを書き下す Plugin）
- 📝 ブログ（二次資料・Anthropic）：サブエージェントがクリーンな文脈で深掘りし凝縮要約（1〜2K トークン）だけ親へ返す「関心の分離」設計 → [[articles/effective-context-engineering-for-ai-agents]]
- 📝 ブログ（二次資料・Anthropic）：長時間タスクで testing/QA/cleanup の専門エージェントへ分業する余地（Future work）→ [[articles/effective-harnesses-for-long-running-agents]]
