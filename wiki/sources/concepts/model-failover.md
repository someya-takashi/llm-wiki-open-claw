---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/model-failover
source_path: raw/docs/concepts/model-failover.md
doc_section: concepts
title: "モデルのフェイルオーバー"
ingested: 2026-06-14
tags: [model-failover, auth-rotation, cooldown, fallback, billing-invalidation]
related:
  - "[[concepts/model-failover]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/oauth]]"
---

# モデルのフェイルオーバー（解説）

> 原典: `raw/docs/concepts/model-failover.md` ・ https://docs.openclaw.ai/ja-JP/concepts/model-failover

## 一言まとめ

失敗を 2 段階で処理：①プロバイダー内の**認証プロファイルローテーション**、②`agents.defaults.model.fallbacks` の**モデルフォールバック**。ランタイム規則とそれを支えるデータを説明する。

## 位置づけ

[[concepts/model-failover]] の中核ソース。[[concepts/model-providers]] の認証設定の上で動き、サブスク認証は [[concepts/oauth]]、HTTP 単位の再試行は [[concepts/retry]]。

## 仕組み・ふるまい

- **ランタイムフロー**：選択ソースポリシー → 認証ストレージ（キー＋OAuth）→ プロファイル ID → ローテーション順。**セッションの固定性**（同セッションは同プロファイルでキャッシュに優しく）。OpenAI Codex サブスク＋API キーバックアップ。
- **クールダウン**：失敗プロファイル/モデルを一定時間スキップ。**請求による無効化**（課金切れ）でクールダウン。
- **モデルフォールバック**：候補チェーンのルール、フォールバックを進めるエラー、継続する/しない条件、クールダウンのスキップとプローブ。
- **セッション上書き**：`/model` でライブモデル切り替え（実行中はクリーンな再試行点で）。可観測性と失敗サマリー。

## 設定・使い方の要点

- `agents.defaults.model.{primary,fallbacks}`、認証プロファイル、クールダウン設定。複数アカウント/モデルを用意して耐障害性を上げる。

## 注意点・落とし穴

- セッション固定性のため、同セッションでは可能な限り同じプロファイルを使う（プロンプトキャッシュ効率）。請求エラーは即クールダウン対象。

## 用語と略称

- **認証プロファイル（auth profile）** = 1 プロバイダーの 1 認証情報
- **クールダウン** = 失敗後に一定時間その候補を避ける
- **請求による無効化** = 課金切れ検知でプロファイルを無効化
- **フォールバックチェーン** = 順に試すモデルの並び

## 関連ページ

- [[concepts/model-failover]] — 対応する概念ページ
- [[concepts/model-providers]] / [[concepts/oauth]] / [[concepts/retry]] / [[concepts/local-models]]
