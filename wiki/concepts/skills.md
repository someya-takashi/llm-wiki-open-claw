---
type: concept
aliases: [Skills, スキル, SKILL.md, AgentSkills]
tags: [skills, skill-md, instructions, gating, allowlist, agentskills]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/slash-commands]]"
components:
  - "[[components/plugin-system]]"
  - "[[components/clawhub]]"
sources:
  - "[[sources/tools/skills]]"
  - "[[sources/tools/creating-skills]]"
  - "[[sources/tools/skills-config]]"
updated: 2026-06-14
---

# Skills

Skills（スキル, エージェントに「やり方」を教える `SKILL.md` 指示パック）は、ツール・Skills・Plugins という拡張の三分法の中段。**ツールが「できること（呼べる関数）」、Skills が「やり方（手順・基準・制約）」、Plugins が「機能そのものの追加」**。各 Skill は YAML frontmatter＋手順を持つディレクトリで、AgentSkills 仕様に互換。

## なぜ重要か

エージェントに必要なツールはあるのに、再現可能なワークフロー・レビュー基準・コマンド順序・運用上の制約を教えたい——そのとき Skill を使う（[[sources/tools/tools]] の判断表）。対象になった Skill はコンパクトな XML として [[concepts/system-prompt]] に注入され、エージェントが「いつ・どう」ツールを使うかを学ぶ。コードを書かずにエージェントの振る舞いを形作れる軽量な拡張点。

## 仕組み（要点）

- **場所と優先順位（高→低）**：`<workspace>/skills` → `<workspace>/.agents/skills` → `~/.agents/skills` → `~/.openclaw/skills` → 同梱 → `skills.load.extraDirs`。同名は最優先が勝つ。
- **可視性は場所と別制御**：`agents.defaults.skills`（ベースライン）＋ `agents.list[].skills`（置換、`[]` で無効）。マルチエージェント（[[concepts/multi-agent]]）では各エージェントが別の許可リストを持てる。
- **ゲート**：`metadata.openclaw.requires`（bins/env/config）・`os`・インストーラー仕様で、環境が満たすときだけ読み込む。
- **スナップショット**：セッション開始時に対象を確定し、ウォッチャーで `SKILL.md` 変更をホットリロード。`user-invocable` な Skill は [[concepts/slash-commands]] として公開され、`command-dispatch: tool` でモデルを介さずツール直送もできる。

詳細・トークン影響は [[sources/tools/skills]]、作成手順は [[sources/tools/creating-skills]]、設定スキーマは [[sources/tools/skills-config]]。

## 既存 wiki とのつながり

Skill は実体としては [[components/plugin-system]] が同梱もでき（Plugin の `skills` ディレクトリ）、公開は [[components/clawhub]]。⚠️ **サードパーティ Skill は信頼できないコード**——`skills.entries.*.env`/`apiKey` は**ホスト**プロセスにシークレットを注入し、サンドボックス化セッションでは効かない（コンテナに別途 env が要る）。信頼できない Skill は [[concepts/sandboxing]] で隔離する、というのが [[concepts/security]]・[[concepts/threat-model]] の帰結。

## 代表ソース

- [[sources/tools/skills]] — 読み込み/優先順位/ゲート/ClawHub/トークン影響
- [[sources/tools/creating-skills]] — `SKILL.md` の作成とテスト
- [[sources/tools/skills-config]] — `skills.*` / `agents.*.skills` 設定スキーマ

## 関連ページ

- [[concepts/session-tool]] / [[concepts/system-prompt]] / [[concepts/slash-commands]]
- [[concepts/sandboxing]] / [[concepts/multi-agent]]
- [[components/plugin-system]] / [[components/clawhub]]
