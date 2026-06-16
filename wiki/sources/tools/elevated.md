---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/elevated
source_path: raw/docs/tools/elevated.md
doc_section: tools
title: "昇格モード（Elevated）"
ingested: 2026-06-14
tags: [elevated, sandbox-escape, exec, directive, allowlist]
related:
  - "[[concepts/exec]]"
  - "[[concepts/sandboxing]]"
  - "[[sources/tools/exec-approvals]]"
---

# 昇格モード（Elevated）（解説）

> 原典: `raw/docs/tools/elevated.md` ・ https://docs.openclaw.ai/ja-JP/tools/elevated

## 一言まとめ

サンドボックス内のエージェントが、**サンドボックスの外に出てコマンドを実行する**ための制御された脱出口。承認ゲートも設定でき、`/elevated` ディレクティブで切り替える。

## 位置づけ

[[concepts/sandboxing]] の「3 制御」のうち**昇格**そのもの（サンドボックス＝どこで、ツールポリシー＝どのツール、昇格＝サンドボックス外し）。[[concepts/exec]] の実行が承認（[[sources/tools/exec-approvals]]）を伴ってホストへ抜ける経路。

## 仕組み・ふるまい

- **ディレクティブ**：`/elevated [on|off|ask|full]`（エイリアス `/elev`）。on でサンドボックス外実行を許可、ask で承認、full で承認スキップ。
- **解決順序**：セッション上書き → エージェント設定 → グローバル。利用可否は許可リストで制御。
- **昇格が制御しないもの**：ツールの可視性（ツールポリシー）やサンドボックスの存在自体は別レイヤー。

## 設定・使い方の要点

- 昇格はあくまで「サンドボックス外し」で、実際の実行可否は Exec 承認（[[sources/tools/exec-approvals]]）が決める。`/elevated full` は YOLO 相当（承認なし）。

## 注意点・落とし穴

- ⚠️ 昇格 exec はサンドボックスの隔離を意図的に破る——`open_groups_with_elevated` のような組み合わせは [[concepts/security]] の監査で警告される高リスク構成。

## 用語と略称

- **昇格モード（elevated）** = サンドボックス外でコマンドを実行する制御された脱出
- **`/elevated`** = 昇格を切り替えるディレクティブ
- **ask / full** = 承認を求める / 承認を外す昇格レベル
- **サンドボックス脱出** = 隔離環境の外で実行すること

## 関連ページ

- [[concepts/exec]] / [[concepts/sandboxing]] / [[sources/tools/exec-approvals]]
- [[sources/tools/exec]] / [[concepts/security]] / [[concepts/slash-commands]]
