---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/llm-task
source_path: raw/docs/tools/llm-task.md
doc_section: tools
title: "LLM Task"
ingested: 2026-06-14
tags: [llm-task, json, structured-output, workflow, plugin, tool]
related:
  - "[[concepts/session-tool]]"
  - "[[sources/tools/lobster]]"
  - "[[components/plugin-system]]"
---

# LLM Task（解説）

> 原典: `raw/docs/tools/llm-task.md` ・ https://docs.openclaw.ai/ja-JP/tools/llm-task

## 一言まとめ

`llm-task` は **JSON のみの LLM ステップ**を実行し、構造化出力（任意で JSON Schema 検証）を返すオプション Plugin ツール。ワークフローに単一の LLM 判断を差し込むのに向く。

## 位置づけ

[[concepts/session-tool]] の Plugin ツール（[[components/plugin-system]]）。[[sources/tools/lobster]] のような決定的ワークフローの各ステップとして、カスタムコードを書かずに LLM 判断を足せる。

## 仕組み・ふるまい

- プロンプト→JSON のみの応答（Schema 検証可）。決定論的フローの中で「分類」「抽出」「判定」などの 1 ステップを担う。

## 設定・使い方の要点

- JSON Schema を渡せば出力を検証。Lobster や TaskFlow の各ステップに組み込む。

## 用語と略称

- **`llm-task`** = JSON 構造化出力を返す LLM ステップツール
- **JSON Schema** = JSON 構造を検証する仕様
- **構造化出力** = 自由文でなく機械可読な出力

## 関連ページ

- [[concepts/session-tool]] / [[sources/tools/lobster]] / [[components/plugin-system]]
- [[sources/automation/taskflow]]
