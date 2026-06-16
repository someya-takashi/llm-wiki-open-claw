---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/context
source_path: raw/docs/concepts/context.md
doc_section: concepts
title: "コンテキスト"
ingested: 2026-06-14
tags: [context, context-window, tokens, compaction, pruning, slash-commands]
related:
  - "[[concepts/context]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/context-engine]]"
---

# コンテキスト（解説）

> 原典: `raw/docs/concepts/context.md` ・ https://docs.openclaw.ai/ja-JP/concepts/context

## 一言まとめ

「コンテキスト」とは **1 回の実行で OpenClaw がモデルへ送るすべて**――システムプロンプト＋会話履歴＋ツール呼び出し/結果＋添付。モデルの**コンテキストウィンドウ（トークン上限）**に制限される。何が含まれ、どう確認し（`/context`）、どう空けるか（`/compact`）を初学者向けに解説したページ。

## 位置づけ

[[concepts/system-prompt]] は「コンテキストの中の固定部分」、本ページは「コンテキスト全体」。何を含めて要約するかを実装として制御するのが [[concepts/context-engine]]、長い履歴の圧縮が [[concepts/compaction]]。**コンテキスト ≠ メモリ**：メモリはディスクに保存して後で再読込できるが、コンテキストは今モデルのウィンドウに載っているもの。

## 仕組み・ふるまい

### コンテキストウィンドウに含まれるもの

システムプロンプト（全セクション）、会話履歴、ツール呼び出し＋結果、添付/文字起こし（画像/音声/ファイル）、Compaction 要約と剪定アーティファクト、プロバイダーの「ラッパー」や非表示ヘッダー（見えなくても含まれる）。

### ツールの「2 種類のコスト」

ツールは①システムプロンプト内の**ツールリストテキスト**と、②モデルへ送る**ツールスキーマ（JSON）**の両方でコンテキストを消費する。スキーマはテキスト表示されないが含まれる。`/context detail` で支配的なスキーマを内訳できる。

### 保持・圧縮・剪定の違い

- **通常履歴**：ポリシーで Compaction/剪定されるまでトランスクリプトに保持。
- **Compaction**：要約をトランスクリプトに残し、最近のメッセージはそのまま維持。
- **剪定（pruning）**：古いツール結果を**メモリ内プロンプトからのみ**削除（トランスクリプトは書き換えない。完全履歴はディスクに残る）。

## 設定・使い方の要点

- 確認コマンド：`/status`（ウィンドウの埋まり具合）、`/context list`（注入物とおおよそのサイズ）、`/context detail`（ファイル/ツールスキーマ/Skills ごとの詳細）、`/context map`（ツリーマップ画像）、`/usage tokens`（返信ごとの使用量フッター）、`/compact`（古い履歴を要約して空ける）。
- `/context` は最新の**実行で構築された**システムプロンプトレポート（`System prompt (run)`）を優先し、無ければその場推定（`estimate`）。完全プロンプトやツールスキーマはダンプしない。
- スラッシュコマンドは Gateway が処理。ディレクティブ（`/think` `/verbose` `/model` `/queue` など）はモデルがメッセージを見る前に取り除かれ、セッション設定を保持する。

## 注意点・落とし穴

- 既定のコンテキストエンジンは組み込み `legacy`。Plugin で `kind:"context-engine"` を入れ `plugins.slots.contextEngine` で選ぶと組み立て・`/compact`・サブエージェントのコンテキストライフサイクルがそのエンジンに委譲される（→ [[concepts/context-engine]]）。`ownsCompaction:false` でも legacy へ自動フォールバックしない。
- Skills 指示は既定で本文に含まれず、モデルが必要時に `SKILL.md` を `read` する前提。Skills **リスト**自体は実コストがある。

## 用語と略称

- **コンテキストウィンドウ** = モデルが一度に扱えるトークン上限
- **剪定（pruning）** = メモリ内プロンプトから古いツール結果を落とす処理（トランスクリプトは不変）
- **ディレクティブ** = モデルに見せず取り除く `/think` 等のセッション設定コマンド
- **tok** = token（トークン、課金/容量の単位）

## 関連ページ

- [[concepts/context]] — 対応する概念ページ
- [[concepts/system-prompt]] / [[concepts/context-engine]]
- [[concepts/session]] / [[concepts/session-pruning]] / [[concepts/compaction]]
