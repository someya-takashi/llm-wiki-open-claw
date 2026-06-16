---
type: concept
aliases: [Agent runtime, エージェントランタイム, Agent process]
tags: [agent, workspace, bootstrap, session, skills]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/agent-workspace]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/soul]]"
  - "[[concepts/multi-agent]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/agent]]"
updated: 2026-06-14
---

# エージェント（単一プロセスの契約）

> ⚠️ **混同注意**：本ページ（`agent`）は **1 つのエージェントプロセスの契約**――どこで動き、何を読み、会話をどこに残すか。ターンを実行する**バックエンドの層**（pi/codex/claude-cli）は [[concepts/agent-runtimes]] を見ること。

OpenClaw は [[components/gateway]] ごとに **単一の埋め込みエージェントランタイム**を 1 プロセスとして走らせる。その 1 プロセスが拠って立つ契約は 3 点に集約できる：

1. **ワークスペース**（`agents.defaults.workspace`）= ツールとコンテキストにとって**唯一の作業ディレクトリ（cwd）**。
2. **ブートストラップファイル**（`AGENTS.md` `SOUL.md` `TOOLS.md` `IDENTITY.md` `USER.md`、初回のみ `BOOTSTRAP.md`）= 新セッション最初のターンでシステムプロンプトに挿入される、人格・手順・ユーザー像。
3. **セッション** = `~/.openclaw/agents/<agentId>/sessions/<id>.jsonl` に **JSONL** で残る安定したトランスクリプト。

## なぜ重要か

この「1 ワークスペース＝1 エージェントの人格と記憶」という単純な契約が、OpenClaw を**設定可能だが予測しやすい**ものにしている。ペルソナや手順をコード変更ではなくワークスペース上の markdown で差し替えられ、Skills も場所の優先順位だけで読み込まれる。実行基盤は **Pi エージェントコア**の上に薄く乗る OpenClaw レイヤーで、ここに [[concepts/agent-loop]] や [[concepts/agent-runtimes]] が接続する。

詳細（Skills の探索パス、ブロックストリーミング、モデル参照解決など）は [[sources/concepts/agent]] を参照。ワークスペースそのものは [[concepts/agent-workspace]]、ブートストラップファイルがプロンプトへ注入される仕組みは [[concepts/system-prompt]]、`SOUL.md` の書き方は [[concepts/soul]]。複数エージェントの分離は [[concepts/multi-agent]]。

## 関連

- [[concepts/agent-runtimes]] — 実行バックエンドの層（混同注意）
- [[concepts/agent-loop]] — 1 ターンの実行配線
- [[components/gateway]]
