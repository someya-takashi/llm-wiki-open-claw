---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/models
source_path: raw/docs/concepts/models.md
doc_section: concepts
title: "モデル（選択と CLI）"
ingested: 2026-06-14
tags: [models, model-selection, cli, registry, allowlist, openrouter-scan]
related:
  - "[[concepts/model-providers]]"
  - "[[concepts/model-failover]]"
  - "[[components/cli]]"
---

# モデル（選択と CLI）（解説）

> 原典: `raw/docs/concepts/models.md` ・ https://docs.openclaw.ai/ja-JP/concepts/models

## 一言まとめ

モデル選択の仕組みと `openclaw models` CLI。`provider/model` の解決順・許可リスト・「モデルは許可されていません」エラーの直し方・モデルレジストリ（`models.json`）を扱う。

## 位置づけ

[[concepts/model-providers]] の選択/管理面（プロバイダー設定の本体は [[sources/concepts/model-providers]]、失敗時の挙動は [[concepts/model-failover]]）。CLI は [[components/cli]]。

## 仕組み・ふるまい

- **モデル選択**：選択元（primary/fallbacks/セッション上書き）とフォールバック動作。クイックモデルポリシー。`agents.defaults.models` 許可リスト。
- **CLI**：`openclaw models list`（設定/認証済み）・`models status`（認証/エンドポイント）。OpenRouter 無料モデルのスキャン。**モデルレジストリ** `models.json`。
- チャットで `/model`（[[concepts/slash-commands]]）切り替え。

## 設定・使い方の要点

- 設定キー概要・安全な許可リスト編集。オンボーディング推奨（`openclaw onboard`）。`agents.defaults.models` の `provider/*` エントリで検出モデルを絞る。

## 注意点・落とし穴

- ⚠️ **「モデルは許可されていません」で返信が止まる**：許可リスト（`agents.defaults.models`）にモデルが無い・認証未設定・プロバイダー未検出が典型原因。`models status` で診断。

## 用語と略称

- **モデル参照（model reference）** = `provider/model` 形式
- **許可リスト（allowlist）** = `agents.defaults.models` で使えるモデルを制限
- **`models.json`** = モデルレジストリ（カタログ）
- **OpenRouter スキャン** = 無料モデルを検出する機能

## 関連ページ

- [[concepts/model-providers]] / [[concepts/model-failover]] / [[components/cli]]
- [[concepts/slash-commands]] / [[concepts/agent-runtimes]]
