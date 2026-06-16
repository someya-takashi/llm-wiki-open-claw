---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/diffs
source_path: raw/docs/tools/diffs.md
doc_section: tools
title: "Diffs"
ingested: 2026-06-14
tags: [diffs, plugin, read-only, artifact, skill, review]
related:
  - "[[concepts/session-tool]]"
  - "[[components/plugin-system]]"
  - "[[sources/tools/apply-patch]]"
---

# Diffs（解説）

> 原典: `raw/docs/tools/diffs.md` ・ https://docs.openclaw.ai/ja-JP/tools/diffs

## 一言まとめ

`diffs` は、変更内容を**エージェント向けの読み取り専用 diff 成果物**に変換するオプション Plugin ツール（短い組み込みガイダンス＋付随 Skill 付き）。

## 位置づけ

[[concepts/session-tool]] の Plugin ツール（[[components/plugin-system]]）。ファイルを実際に書き換える [[sources/tools/apply-patch]] と対で、変更の**レビュー/提示**を担う。Skill（[[concepts/skills]]）が使い方を教える。

## 仕組み・ふるまい

- ファイルや Markdown の差分を受け取り、レビューしやすい読み取り専用の diff 成果物としてレンダリング（複数の入力形式に対応）。

## 設定・使い方の要点

- 変更を適用する前にエージェント/人間がレビューする用途。付随 Skill が呼び出しパターンを案内。

## 用語と略称

- **diff** = 変更前後の差分
- **読み取り専用成果物** = 適用せず提示するだけの出力
- **Skill** = ツールの使い方を教える指示パック（[[concepts/skills]]）

## 関連ページ

- [[concepts/session-tool]] / [[components/plugin-system]] / [[sources/tools/apply-patch]]
- [[concepts/skills]]
