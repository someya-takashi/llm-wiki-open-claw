---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/experimental-features
source_path: raw/docs/concepts/experimental-features.md
doc_section: concepts
title: "実験的な機能"
ingested: 2026-06-14
tags: [experimental, flags, local-model, opt-in]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/context]]"
  - "[[concepts/agent]]"
---

# 実験的な機能（解説）

> 原典: `raw/docs/concepts/experimental-features.md` ・ https://docs.openclaw.ai/ja-JP/concepts/experimental-features

## 一言まとめ

OpenClaw の実験的機能は、明示的なフラグの背後に置かれた**オプトインのプレビューサーフェス**。安定デフォルトや長期の公開契約に値する前段階で、形状・挙動が速く変わることを前提に扱う、という位置づけと現行フラグの一覧。

## 位置づけ

通常の設定とは別物として扱う「プレビュー棚」。`agents.defaults.experimental.*` や `tools.experimental.*` といった**専用の名前空間**に置かれ、安定したノブ（[[concepts/agent]] のツールポリシー等）に紛れ込ませない方針。各フラグは個別の機能ページ（ローカルモデル、メモリ検索、計画ツール）に紐づく。

## 仕組み・ふるまい（現行フラグ）

- `agents.defaults.experimental.localModelLean`（ローカルモデルのリーンモード）— 後述。
- `agents.defaults.memorySearch.experimental.sessionMemory` — `memory_search` で過去セッションのトランスクリプトを索引化（追加のストレージ/索引コストを許容する場合）。
- `tools.experimental.planTool` — 互換ランタイム/UI で複数ステップ作業追跡用の構造化 `update_plan` ツールを公開。

### ローカルモデルのリーンモード

`localModelLean: true` は弱いローカルモデル構成向けの**圧力逃がし弁**。有効にすると毎ターン、ツールサーフェスから説明が長くパラメーター形状が多い 3 ツール **`browser`・`cron`・`message`** を削除する（それ以外は不変）。小さなコンテキストや厳格な OpenAI 互換バックエンドでは、ツールスキーマがプロンプトに収まるか・モデルが正しいツールを選べるか・Chat Completions アダプターが構造化出力制限内に収まるかの差になる。削除しても暗黙の再配線は無く、`read`/`write`/`edit`/`exec`/`apply_patch`・Web 検索/取得・メモリ・セッション/エージェントツールは引き続き使える。

## 設定・使い方の要点

- リーンモードを使う場面：モデルが Gateway と通信できるのは確認済みだが完全なエージェントターンが不安定なとき。手順例：①`openclaw infer model run --gateway --model <ref> --prompt "Reply with exactly: pong"` 成功 → ②通常ターンで不正なツール呼び出し/過大プロンプト等が発生 → ③`localModelLean: true` で解消。
- 有効化後は Gateway を再起動し、`openclaw status --deep` で `browser`/`cron`/`message` が消えていることを確認。

## 注意点・落とし穴

- **実験的＝隠す、ではない**。実験機能はドキュメントと設定パス自体で明示すべきで、安定して見えるデフォルトにプレビュー挙動を紛れ込ませない。
- リーンモードは回避策であってデフォルトではない。恒久的に狭いツールサーフェスが要るなら `tools.profile`・`tools.allow`/`deny`・モデルの `compat.supportsTools: false` といった**安定ノブ**を優先する。

## 用語と略称

- **オプトイン** = 既定オフで、明示的に有効化したときだけ働く
- **リーンモード** = ツールサーフェスを意図的に削った軽量構成
- **Chat Completions アダプター** = OpenAI 互換 API へ橋渡しする層

## 関連ページ

- [[concepts/agent-runtimes]] — ローカル/セルフホストのバックエンド
- [[concepts/context]] — ツールスキーマがコンテキストを食う問題
- [[concepts/agent]] — 安定したツールポリシー（profile/allow/deny）
