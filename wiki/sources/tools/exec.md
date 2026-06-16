---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/exec
source_path: raw/docs/tools/exec.md
doc_section: tools
title: "Exec ツール"
ingested: 2026-06-14
tags: [exec, process, shell, command, host, sandbox, allowlist]
related:
  - "[[concepts/exec]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/session-tool]]"
---

# Exec ツール（解説）

> 原典: `raw/docs/tools/exec.md` ・ https://docs.openclaw.ai/ja-JP/tools/exec

## 一言まとめ

ワークスペースで**シェルコマンドを実行する** `exec` ツール。`process` でフォア/バックグラウンド実行に対応。変更を伴うシェルサーフェスで、選択ホスト/サンドボックスが許す場所ならファイルを作成/編集/削除できる。

## 位置づけ

[[concepts/exec]] の中核ツール。実行場所と承認は [[concepts/sandboxing]]（サンドボックス/ツールポリシー/昇格の 3 制御）と [[sources/tools/exec-approvals]]、エージェントツールとしての位置は [[concepts/session-tool]]。

## 仕組み・ふるまい

- パラメーター：コマンド・`yieldMs`/`background`（`process` 許可時）。`process` 不許可なら同期実行で `yieldMs`/`background` を無視。バックグラウンドセッションはエージェントごとにスコープ。
- **認可モデル**：ツールポリシー → 昇格ゲート（[[sources/tools/elevated]]）→ Exec 承認（[[sources/tools/exec-approvals]]）の重ね合わせ。実行ホストは `host=auto|sandbox|gateway|node`。
- Allowlist＋safe bins で許可コマンドを絞る。`apply_patch` 相当のファイル編集も含む。

## 設定・使い方の要点

- 設定 `tools.exec.*`（`host`/`security`/`ask`/`node`）、PATH の扱い、セッション上書き `/exec host=... security=... ask=... node=...`（[[concepts/slash-commands]]）。

## 注意点・落とし穴

- ⚠️ **`exec` は読み取り専用にならない**：`write`/`edit`/`apply_patch` を無効化しても `exec` でファイル変更できる。`exec` に到達できる経路は「変更可能なシェル面」とみなす（[[concepts/threat-model]]・[[concepts/http-api]] のハード拒否）。

## 用語と略称

- **exec / process** = シェル実行ツール / フォア・バックグラウンド管理
- **safe bins** = stdin のみで安全に実行できる許可バイナリ
- **allowlist** = 許可するコマンドの明示リスト
- **host=node** = ノードホストでコマンドを実行する指定

## 関連ページ

- [[concepts/exec]] — 対応する概念ページ
- [[sources/tools/exec-approvals]] / [[sources/tools/elevated]] / [[sources/tools/apply-patch]]
- [[concepts/sandboxing]] / [[concepts/session-tool]] / [[components/node]]
