---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/trajectory
source_path: raw/docs/tools/trajectory.md
doc_section: tools
title: "軌跡バンドル（Trajectory）"
ingested: 2026-06-14
tags: [trajectory, flight-recorder, export, jsonl, support-bundle, diagnostics]
related:
  - "[[concepts/session]]"
  - "[[concepts/diagnostics]]"
  - "[[concepts/slash-commands]]"
---

# 軌跡バンドル（Trajectory）（解説）

> 原典: `raw/docs/tools/trajectory.md` ・ https://docs.openclaw.ai/ja-JP/tools/trajectory

## 一言まとめ

Trajectory キャプチャは**セッションごとのフライトレコーダー**。各エージェント実行の構造化タイムラインを記録し、`/export-trajectory` が現在のセッションを秘匿化されたサポートバンドルにパッケージ化する。

## 位置づけ

[[concepts/session]] の実行記録で、[[concepts/diagnostics]]（health/doctor/診断 zip）のセッション版。エクスポートは [[concepts/slash-commands]] の `/export-trajectory`（exec 承認を要求）。

## 仕組み・ふるまい

- プロンプト・ツール・トランスクリプトのタイムラインを JSONL で記録。`/export-trajectory [path]` が秘匿化済みバンドルを出力（グループでは承認/結果をオーナーに非公開送信）。

## 設定・使い方の要点

- 「あのとき何が起きたか」を 1 セッション分追う必要があるとき（バグ報告・挙動調査）に使う。

## 注意点・落とし穴

- `/export-trajectory` は exec 承認を求める——「すべて許可」で通さない（[[concepts/security]]）。

## 用語と略称

- **Trajectory（軌跡）** = エージェント実行の構造化タイムライン
- **フライトレコーダー** = 起きたことを記録する仕組み
- **JSONL** = JSON Lines（1 行 1 レコード）
- **サポートバンドル** = 調査用にまとめた秘匿化済みアーカイブ

## 関連ページ

- [[concepts/session]] / [[concepts/diagnostics]] / [[concepts/slash-commands]]
- [[concepts/security]]
