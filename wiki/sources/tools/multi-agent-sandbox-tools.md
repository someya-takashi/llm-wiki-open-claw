---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/multi-agent-sandbox-tools
source_path: raw/docs/tools/multi-agent-sandbox-tools.md
doc_section: tools
title: "マルチエージェントのサンドボックスとツール"
ingested: 2026-06-14
tags: [multi-agent, sandbox, tool-policy, per-agent, override, precedence]
related:
  - "[[concepts/multi-agent]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/delegate-architecture]]"
---

# マルチエージェントのサンドボックスとツール（解説）

> 原典: `raw/docs/tools/multi-agent-sandbox-tools.md` ・ https://docs.openclaw.ai/ja-JP/tools/multi-agent-sandbox-tools

## 一言まとめ

マルチエージェント構成で、**各エージェントがグローバルなサンドボックス/ツールポリシーを上書き**する方法。エージェントごとの設定・優先順位ルール・移行例を示す。

## 位置づけ

[[concepts/multi-agent]] と [[concepts/sandboxing]] の交点。委任エージェントに権限ティアを与える [[concepts/delegate-architecture]] の実装面で、「読み取り専用エージェント」「通信のみエージェント」などを作る。

## 仕組み・ふるまい

- **サンドボックス設定の優先**：`agents.list[].sandbox` がグローバル `agents.defaults.sandbox`/`tools.sandbox` を上書き。
- **ツール制限**：`agents.list[].tools.allow`/`deny` でエージェントごとに絞る。例：読み取り専用（fs 書き込み無効）、ファイルシステム無効＋シェル実行、通信のみ。

## 設定・使い方の要点

- 単一エージェントからの移行：`tools.*`/`sandbox.*` を `agents.list[].*` へ。⚠️ よくある落とし穴「non-main」——メインでないエージェントの設定漏れに注意。テスト/トラブルシュート手順あり。

## 注意点・落とし穴

- エージェントごとの権限を狭めることが委任の安全性の核（[[concepts/security]]）。優先順位（グローバル→エージェント）を取り違えると意図せず広い権限になる。

## 用語と略称

- **エージェントごと上書き（per-agent override）** = `agents.list[]` 単位の設定
- **優先順位（precedence）** = グローバル → エージェント設定の適用順
- **読み取り専用エージェント** = 書き込みツールを外したエージェント
- **non-main** = メイン以外のエージェント（設定漏れの落とし穴）

## 関連ページ

- [[concepts/multi-agent]] / [[concepts/sandboxing]] / [[concepts/delegate-architecture]]
- [[concepts/exec]] / [[concepts/security]] / [[sources/gateway/config-agents]]
