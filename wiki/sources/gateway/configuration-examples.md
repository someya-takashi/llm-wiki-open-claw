---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/configuration-examples
source_path: raw/docs/gateway/configuration-examples.md
doc_section: gateway
title: "設定例"
ingested: 2026-06-14
tags: [configuration, examples, recipes, json5]
related:
  - "[[concepts/configuration]]"
  - "[[sources/gateway/configuration]]"
---

# 設定例（解説）

> 原典: `raw/docs/gateway/configuration-examples.md` ・ https://docs.openclaw.ai/ja-JP/gateway/configuration-examples

## 一言まとめ

現行スキーマに沿った**コピー＆ペースト可能な `openclaw.json` 例**集。絶対最小構成から、用途別パターン（マルチプラットフォーム・セキュア DM・制限付き仕事用ボット・ローカルモデルのみ等）まで。

## 位置づけ

[[concepts/configuration]] を「動く具体例」で示すページ。フィールドの意味は [[sources/gateway/configuration]]、網羅辞書は [[sources/gateway/configuration-reference]]。初めての設定は `openclaw onboard` か本ページの例から始めるのが早い。

## 仕組み・ふるまい（収録パターン）

- **クイックスタート**：絶対最小構成（workspace＋1 チャネルの allowFrom）／推奨スターター。
- **展開例（主要オプション）**：主要キーを一通り盛り込んだ参照用の大きな例。
- **一般的なパターン**：
  - **共有スキルベースライン＋1 つの上書き**（`agents.defaults.skills` ＋ `agents.list[].skills`）。
  - **マルチプラットフォーム設定**（複数チャネル同時）。
  - **信頼済みノードネットワークの自動承認**（同一 tailnet などの自動ペアリング）。
  - **セキュア DM モード**（共有受信箱/マルチユーザー DM。`session.dmScope: "per-channel-peer"` 等でユーザー分離）。
  - **Anthropic API キー ＋ MiniMax フォールバック**（`model.primary`＋`fallbacks`）。
  - **仕事用ボット（制限付きアクセス）**（ツール allow/deny ＋ サンドボックス）。
  - **ローカルモデルのみ**（self-hosted/ollama 構成）。
  - シンボリックリンクされた兄弟スキルリポジトリ。

## 設定・使い方の要点

- **セキュア DM モード**は複数人が DM する環境で必須級（DM 分離をしないと他人の私信が見える → [[concepts/session]]）。
- **制限付きボット**はエージェントごとの `tools.allow/deny` ＋ [[concepts/sandboxing]] を Gateway レベルで効かせる（[[concepts/multi-agent]] / [[concepts/delegate-architecture]] の実装例にあたる）。
- 大きな設定は [[sources/gateway/configuration]] の `$include` で分割すると管理しやすい。

## 注意点・落とし穴

- 例はスキーマ変更で古くなり得る。貼り付け後に `openclaw config validate` と `openclaw doctor` で検証する。
- API キー等はリテラル直書きでなく `${VAR}`/SecretRef を使う（[[sources/gateway/configuration]] の環境変数節）。

## 用語と略称

- **スターター（starter）** = 最初に置く推奨ベース設定
- **自動承認（auto-approve）** = 信頼ネットワークでペアリングを自動許可する設定
- **フォールバック（fallback）** = 主モデル失敗時に切り替える代替モデル

## 関連ページ

- [[concepts/configuration]] — 対応する概念ページ
- [[sources/gateway/configuration]] / [[sources/gateway/configuration-reference]]
- [[concepts/multi-agent]] / [[concepts/session]] / [[concepts/sandboxing]]
