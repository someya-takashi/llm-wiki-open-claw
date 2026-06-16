---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/thinking
source_path: raw/docs/tools/thinking.md
doc_section: tools
title: "思考レベル（Thinking）"
ingested: 2026-06-14
tags: [thinking, reasoning-effort, directive, model, session]
related:
  - "[[concepts/slash-commands]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/session]]"
---

# 思考レベル（Thinking）（解説）

> 原典: `raw/docs/tools/thinking.md` ・ https://docs.openclaw.ai/ja-JP/tools/thinking

## 一言まとめ

`/think`（`/t`/`/thinking`）で**モデルの思考（reasoning）の深さ**を設定するインラインディレクティブ。`off`/`minimal`/`low`/`medium`/`high` などのレベルをセッション単位で切り替える。

## 位置づけ

[[concepts/slash-commands]] のディレクティブの一つ。レベルはアクティブモデルのプロバイダープロファイルから取り、[[concepts/agent-runtimes]] のプロバイダーごとに対応が異なる。設定先は [[concepts/session]]（ディレクティブのみのメッセージは永続化）。

## 仕組み・ふるまい

- インラインディレクティブ `/t <level>`/`/think:<level>`/`/thinking <level>`。一般レベルは `off`/`minimal`/`low`/`medium`/`high`、`xhigh`/`adaptive`/`max`/`on` などはサポート時のみ。
- レベルはセッション上書きとして効き、`/think default` でクリア。

## 設定・使い方の要点

- 難しいタスクは高レベル、軽い応答は低レベルでコスト/レイテンシを抑える。プロバイダーが対応する範囲のレベルのみ有効。

## 注意点・落とし穴

- `/reasoning`（思考の**表示**）とは別物——`/think` は思考の**量**。グループでは `/reasoning` を晒さない（[[sources/tools/slash-commands]]）。

## 用語と略称

- **思考レベル / reasoning effort** = モデルが内部推論にかける量
- **ディレクティブ** = モデルに見える前に剥がされる `/...` 設定
- **`/think` / `/t`** = 思考レベルを設定するコマンド

## 関連ページ

- [[concepts/slash-commands]] / [[concepts/agent-runtimes]] / [[concepts/session]]
- [[sources/tools/slash-commands]]
