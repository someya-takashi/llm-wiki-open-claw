---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/skills
source_path: raw/docs/tools/skills.md
doc_section: tools
title: "Skills"
ingested: 2026-06-14
tags: [skills, skill-md, agentskills, gating, allowlist, clawhub, precedence]
related:
  - "[[concepts/skills]]"
  - "[[components/clawhub]]"
  - "[[concepts/sandboxing]]"
---

# Skills（解説）

> 原典: `raw/docs/tools/skills.md` ・ https://docs.openclaw.ai/ja-JP/tools/skills

## 一言まとめ

Skills は **エージェントにツールの使い方を教える `SKILL.md` 指示パック**（AgentSkills 互換）。各 Skill は YAML frontmatter＋手順を持つディレクトリで、OpenClaw は同梱＋ローカル上書きを読み込み、環境・設定・バイナリの有無で読み込み時にフィルタ（ゲート）する。

## 位置づけ

[[concepts/skills]] の中核ソース。ツール（[[concepts/session-tool]]）が「できること」、Skills が「やり方」、Plugins（[[components/plugin-system]]）が「機能追加」という三分法（[[sources/tools/tools]]）の中段。配布は [[components/clawhub]]、信頼できない Skill の扱いは [[concepts/sandboxing]]。

## 仕組み・ふるまい

- **場所と優先順位（高→低）**：`<workspace>/skills` → `<workspace>/.agents/skills` → `~/.agents/skills` → `~/.openclaw/skills`（管理対象/ローカル）→ 同梱 → `skills.load.extraDirs`。同名は最優先ソースが勝つ。
- **許可リスト（可視性は場所と別制御）**：`agents.defaults.skills` をベースに、`agents.list[].skills` は**置換**（マージしない）、`[]` で skill なし。
- **ゲート（`metadata.openclaw`）**：`requires.bins`（PATH 必須）/`requires.env`/`requires.config`/`os`/`alwaysInclude`、インストーラー仕様（brew/node/go/uv/download）。条件を満たす skill のみが対象に。
- **スナップショット**：セッション開始時に対象 skill を確定（同セッションで再利用）。Skills ウォッチャー（`skills.load.watch`）で `SKILL.md` 変更をホットリロード。

## 設定・使い方の要点

- `SKILL.md` frontmatter：`name`/`description`（必須、単一行）、`user-invocable`（スラッシュコマンド化）、`command-dispatch: tool`（モデルを介さずツール直送）。手順内のパスは `{baseDir}`。
- ClawHub：`openclaw skills install <slug>`（ワークスペース `skills/` に）/`update --all`。Plugin も `openclaw.plugin.json` の `skills` で同梱可能（低優先）。Skill Workshop Plugin はエージェント作業から skill を生成（既定無効）。
- トークン影響：対象 skill のコンパクト XML をシステムプロンプトに注入（基本 195 文字＋skill ごと 97 文字＋フィールド長）。

## 注意点・落とし穴

- ⚠️ **サードパーティ skill は信頼できないコード**として扱い、有効化前に読む。`skills.entries.*.env`/`apiKey` は**ホスト**プロセスにシークレットを注入（サンドボックス外）。サンドボックス化セッションでは `requires.bins` がコンテナ内にも必要（`sandbox.docker.setupCommand`）。
- ワークスペース検出は realpath がルート内の `SKILL.md` のみ受理（パストラバーサル防止）。シンボリックリンクは `allowSymlinkTargets` で明示。

## 用語と略称

- **Skill** = `SKILL.md` を持つ指示パック（やり方を教える）
- **AgentSkills** = Skill フォーマットの互換仕様（agentskills.io）
- **ゲート（gating）** = 環境/設定/バイナリ条件での読み込み時フィルタ
- **`{baseDir}`** = skill フォルダのパスを差すテンプレート変数
- **ClawHub** = 公開 Skill/Plugin レジストリ

## 関連ページ

- [[concepts/skills]] — 対応する概念ページ
- [[sources/tools/creating-skills]] / [[sources/tools/skills-config]] / [[sources/tools/slash-commands]]
- [[components/clawhub]] / [[components/plugin-system]] / [[concepts/sandboxing]]
