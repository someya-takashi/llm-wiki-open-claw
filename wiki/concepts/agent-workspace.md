---
type: concept
aliases: [Agent workspace, エージェントのワークスペース, workspace]
tags: [workspace, bootstrap, memory, sandbox, git]
related:
  - "[[concepts/agent]]"
  - "[[concepts/soul]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/sandboxing]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/agent-workspace]]"
updated: 2026-06-14
---

# エージェントのワークスペース

**ワークスペース**は、エージェントの「ホーム」であり**唯一の作業ディレクトリ（cwd）**――非公開メモリとして扱う 1 つのディレクトリに、人格・手順・ユーザー像・記憶を markdown で置く。既定は `~/.openclaw/workspace`、`agents.defaults.workspace` で上書き。

## なぜ重要か

OpenClaw の「設定可能だが予測しやすい」性質の源。ペルソナや手順（`SOUL.md`/`AGENTS.md`/`USER.md`/`IDENTITY.md`/`TOOLS.md` と、`memory/` `MEMORY.md`）を**コードでなくワークスペースのファイルで**差し替えられ、それらが [[concepts/system-prompt]] を通じて毎ターン注入される。設定・認証・セッションを置く `~/.openclaw/` とは明確に分離され、ワークスペースは private git リポジトリでバックアップする運用が推奨される。

## 押さえる点

- **強固なサンドボックスではない**：相対パスはワークスペース基準だが、分離が要るなら [[concepts/sandboxing]]（`agents.defaults.sandbox`）を使う。
- **アクティブは 1 つだけ**にする（複数残すと認証/状態がずれる。`openclaw doctor` が警告）。
- **秘密情報をコミットしない**（OAuth トークン等は `~/.openclaw/` 側 → [[concepts/oauth]]）。

詳細なファイルマップ・Git 手順・移行は [[sources/concepts/agent-workspace]]。

## 関連

- [[concepts/agent]] — このワークスペースを使う 1 プロセスの契約
- [[concepts/soul]] — `SOUL.md` の書き方
- [[concepts/system-prompt]] — ファイルが注入される仕組み
- 📝 ブログ（二次資料・Anthropic）：複数コンテキストウィンドウをまたぐ長時間タスクで、進捗ファイル＋git 履歴を「セッション間の橋渡し成果物」として残すハーネス → [[articles/effective-harnesses-for-long-running-agents]]
