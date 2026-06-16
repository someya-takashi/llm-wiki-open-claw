---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/creating-skills
source_path: raw/docs/tools/creating-skills.md
doc_section: tools
title: "Skills の作成"
ingested: 2026-06-14
tags: [skills, skill-md, authoring, workspace, tutorial]
related:
  - "[[concepts/skills]]"
  - "[[sources/tools/skills]]"
  - "[[components/clawhub]]"
---

# Skills の作成（解説）

> 原典: `raw/docs/tools/creating-skills.md` ・ https://docs.openclaw.ai/ja-JP/tools/creating-skills

## 一言まとめ

`SKILL.md` でカスタムワークスペース Skill を作ってテストする入門。ディレクトリを作り frontmatter＋手順を書き、`/new` で読み込んで `openclaw agent --message "..."` で試す、という最小フロー。

## 位置づけ

[[concepts/skills]] の作成者向け（読み込み/優先順位の仕組みは [[sources/tools/skills]]、設定スキーマは [[sources/tools/skills-config]]）。共有は [[components/clawhub]]。

## 仕組み・ふるまい

1. `mkdir -p <workspace>/skills/hello-world` → 2. `SKILL.md`（frontmatter `name`/`description` ＋ markdown 手順）を書く（`name` はハイフンケース、フォルダ名と一致）→ 3. `/new` か `openclaw gateway restart` で読み込み → 4. `openclaw skills list` で確認 → 5. トリガーするメッセージで `openclaw agent --message "..."` テスト。

## 設定・使い方の要点

- frontmatter：`name`/`description`（必須）、`metadata.openclaw.os`/`requires.bins`/`requires.config`（任意ゲート）。場所と優先順位は [[sources/tools/skills]] の表に同じ。
- ベストプラクティス：簡潔に「何をするか」を書く・`exec` を使うなら信頼できない入力からのコマンドインジェクションを許さないプロンプトに・共有前にローカルテスト。

## 注意点・落とし穴

- skill が読み込まれない時は新セッション（`/new`）か Gateway 再起動。フォルダ名と `name` の不一致に注意。

## 用語と略称

- **frontmatter** = `SKILL.md` 冒頭の YAML メタデータ
- **ハイフンケース** = `hello-world` のような小文字＋ハイフン命名
- **ワークスペース skill** = `<workspace>/skills/` 配下の最優先 skill

## 関連ページ

- [[concepts/skills]] / [[sources/tools/skills]] / [[sources/tools/skills-config]]
- [[components/clawhub]] / [[components/plugin-system]]
