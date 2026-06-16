---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/exec-approvals
source_path: raw/docs/tools/exec-approvals.md
doc_section: tools
title: "Exec 承認"
ingested: 2026-06-14
tags: [exec-approvals, allowlist, yolo, security, gateway, node]
related:
  - "[[concepts/exec]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/security]]"
---

# Exec 承認（解説）

> 原典: `raw/docs/tools/exec-approvals.md` ・ https://docs.openclaw.ai/ja-JP/tools/exec-approvals

## 一言まとめ

サンドボックス化エージェントが**実ホスト（gateway/node）でコマンドを実行する**ためのコンパニオンアプリ/ノードホストのガードレール。コマンドは**ポリシー＋許可リスト＋（任意の）ユーザー承認がすべて一致**したときだけ許可される。

## 位置づけ

[[concepts/exec]] の承認システム。ツールポリシーと昇格ゲート（[[sources/tools/elevated]]）の**上に重なる**安全インターロック（昇格が `full` なら承認をスキップ）。[[concepts/sandboxing]]・[[concepts/security]] の中核防御。

## 仕組み・ふるまい

- **適用場所**：`host=gateway`/`host=node` の実行に対し、承認は実行ホスト側（`~/.openclaw/exec-approvals.json`）。信頼モデルと macOS の分離。
- **ポリシー**：`exec.security`（`deny`/`allowlist`/`full`）・`exec.ask`（`off`/`on-miss`/`always`）・`askFallback`・`strictInlineEval`・`commandHighlighting`。
- **エージェントごとの allowlist**：`argPattern` で引数を制限。`openclaw approvals allowlist add --node <id> "/usr/bin/uname"`。

## 設定・使い方の要点

- 有効ポリシー確認 `openclaw approvals get`。**YOLO モード**（承認なし）＝`security: "full"`＋`ask: "off"`（永続「プロンプトなし」やローカルショートカット、ノードホスト、セッション専用ショートカットあり）。
- セッション上書きは `/exec`（[[concepts/slash-commands]]）。承認は正規 `systemRunPlan` にバインドされ承認後の編集を拒否（[[sources/nodes/troubleshooting]]）。

## 注意点・落とし穴

- ⚠️ **YOLO モードは承認を完全に外す**——信頼できる隔離環境のみ。`/diagnostics` 等の都度承認を「すべて許可」で通さない（[[concepts/threat-model]] の exec 承認バイパス）。高度な safe bins / インタープリタ束縛 / 承認転送は [[sources/tools/exec-approvals-advanced]]。

## 用語と略称

- **Exec 承認** = 実ホスト実行を許可する多段ゲート
- **allowlist / deny / full** = 許可リスト / 拒否 / 無制限のセキュリティモード
- **YOLO モード** = 承認を外した無制限実行
- **systemRunPlan** = 承認時に固定される正規の実行計画

## 関連ページ

- [[concepts/exec]] — 対応する概念ページ
- [[sources/tools/exec]] / [[sources/tools/exec-approvals-advanced]] / [[sources/tools/elevated]]
- [[concepts/sandboxing]] / [[concepts/security]] / [[components/node]]
