---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/loop-detection
source_path: raw/docs/tools/loop-detection.md
doc_section: tools
title: "ツールループ検出"
ingested: 2026-06-14
tags: [loop-detection, guardrail, repeated-tool-calls, safety]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/retry]]"
  - "[[concepts/agent-loop]]"
---

# ツールループ検出（解説）

> 原典: `raw/docs/tools/loop-detection.md` ・ https://docs.openclaw.ai/ja-JP/tools/loop-detection

## 一言まとめ

反復的なツール呼び出しパターンを止める 2 つのガードレール。**ループ検出**（`tools.loopDetection`、既定無効）がツール呼び出し履歴を監視し、反復パターンや不明ツール再試行を検出する。

## 位置づけ

[[concepts/session-tool]]・[[concepts/agent-loop]] の暴走防止。一時障害の [[concepts/retry]] とは別で、こちらは「同じことを繰り返すモデル」を止める。

## 仕組み・ふるまい

- ローリング形式のツール呼び出し履歴を監視し、反復パターン・不明ツールの再試行を検出。協調する 2 つのガードレールで構成。

## 設定・使い方の要点

- `tools.loopDetection.enabled`（既定オフ）で有効化。ツール多用エージェントで無限ループ的挙動を抑える。

## 注意点・落とし穴

- 既定無効——必要に応じて有効化。正当な反復作業を誤検知しないよう閾値に注意。

## 用語と略称

- **ループ検出（loop detection）** = 反復ツール呼び出しを検知するガードレール
- **ローリング履歴** = 直近の呼び出しを保持する窓
- **ガードレール（guardrail）** = 暴走を防ぐ安全装置

## 関連ページ

- [[concepts/session-tool]] / [[concepts/agent-loop]] / [[concepts/retry]]
