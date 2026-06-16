---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/exec-approvals-advanced
source_path: raw/docs/tools/exec-approvals-advanced.md
doc_section: tools
title: "Exec 承認 — 高度"
ingested: 2026-06-14
tags: [exec-approvals, safe-bins, interpreter, approval-forwarding, advanced]
related:
  - "[[concepts/exec]]"
  - "[[sources/tools/exec-approvals]]"
  - "[[concepts/sandboxing]]"
---

# Exec 承認 — 高度（解説）

> 原典: `raw/docs/tools/exec-approvals-advanced.md` ・ https://docs.openclaw.ai/ja-JP/tools/exec-approvals-advanced

## 一言まとめ

Exec 承認の高度トピック：`safeBins` 高速パス・インタープリタ/ランタイムのバインド・**チャットチャネルへの承認転送**（ネイティブ配信含む）。中核フローは [[sources/tools/exec-approvals]]。

## 位置づけ

[[concepts/exec]] の詳細面。承認をオペレーターの DM/チャネルに転送して許可する仕組みで、[[concepts/messages]] の配信と [[concepts/pairing]] の信頼に乗る。

## 仕組み・ふるまい

- **Safe bins（stdin のみ）**：argv 検証と拒否フラグ、信頼されたバイナリディレクトリ、シェル連結/ラッパー/マルチプレクサの扱い。許可リストとの比較で使い分け。
- **インタープリタ/ランタイムコマンド**：python 等の runtime を具体ファイルにバインド（フォローアップ配信動作）。
- **承認転送**：Plugin 承認の転送、任意チャネルでの同一チャット承認、**ネイティブ承認配信**（Discord/Telegram の UI、macOS IPC フロー）。

## 設定・使い方の要点

- safe bins は「stdin のみで安全に実行できる」高速パス。許可リストはコマンド全体を承認、safe bins は引数なしの安全実行を許す——両者を組み合わせる。

## 注意点・落とし穴

- ⚠️ シェルラッパー（`bash -c` 等）は safe bins を回避しうるため argv 検証で拒否フラグを弾く。承認転送でオーナー以外に承認を許さない（[[concepts/security]]）。

## 用語と略称

- **safe bins** = stdin のみで安全実行できる許可バイナリ
- **argv 検証** = コマンド引数配列の検査
- **承認転送（approval forwarding）** = 承認要求をチャットに送って許可する仕組み
- **IPC** = Inter-Process Communication（macOS の承認連携）

## 関連ページ

- [[concepts/exec]] / [[sources/tools/exec-approvals]] / [[sources/tools/exec]]
- [[concepts/sandboxing]] / [[concepts/messages]] / [[concepts/security]]
