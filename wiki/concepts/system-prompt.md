---
type: concept
aliases: [System prompt, システムプロンプト]
tags: [system-prompt, prompt, bootstrap, cache, prompt-mode]
related:
  - "[[concepts/context]]"
  - "[[concepts/agent-workspace]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/soul]]"
sources:
  - "[[sources/concepts/system-prompt]]"
updated: 2026-06-14
---

# システムプロンプト

OpenClaw は **エージェント実行ごとに自前のシステムプロンプトを組み立てて注入する**（基盤エージェントの既定プロンプトは使わない）。固定セクション（ツール・実行バイアス・安全性・Skills・ワークスペース・ランタイム等）＋ **Project Context に注入されたワークスペースのブートストラップファイル**で構成される。

## なぜ重要か

「モデルが何を前提に動くか」を OpenClaw が完全に所有する、という設計の核。3 レイヤーの組み立て（純粋レンダラー → 設定解決 → ランタイムのライブ情報収集）で、デバッグ表示とライブ実行を一致させる。さらに **プロンプトキャッシュ境界**の上に安定コンテンツ（Project Context）を、下に揮発的なチャネル/セッションセクションを置くことで、ローカルバックエンドが安定プレフィックスを再利用でき、コストを抑えられる。

## 押さえる点

- 注入されるブートストラップ：`AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md`（＋新規 `BOOTSTRAP.md`、存在すれば `MEMORY.md`）。上限は `bootstrapMaxChars`/`bootstrapTotalMaxChars`。
- **promptMode**：`full`（既定）/`minimal`（サブエージェント）/`none`。
- 安全性ガードレールは**助言**であって強制ではない（強制はツールポリシー・exec 承認・サンドボックス）。

詳細・全セクション・スナップショット検証は [[sources/concepts/system-prompt]]。

## 位置づけ

- [[concepts/context]] の中の最大の固定要素。
- 中身は [[concepts/agent-workspace]]（[[concepts/soul]] 含む）由来。
- [[concepts/agent-loop]] の「プロンプト組み立て」段で確定する。

## 関連

- [[concepts/context]] / [[concepts/agent-workspace]] / [[concepts/agent-loop]]
- 📝 ブログ（二次資料・外部）：精選メモリを system prompt へ注入する（構造化＝YAML frontmatter / 非構造化＝Markdown）パターンと明示区切りでのガードレール → [[articles/context-engineering-personalization]]
- 📝 ブログ（二次資料・Anthropic）：システムプロンプトの「適切な高度（right altitude）」と最小だが十分なセクション設計の総論 → [[articles/effective-context-engineering-for-ai-agents]]
