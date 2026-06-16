---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/retry
source_path: raw/docs/concepts/retry.md
doc_section: concepts
title: "再試行ポリシー"
ingested: 2026-06-14
tags: [retry, backoff, rate-limit, http, provider, channel]
related:
  - "[[concepts/retry]]"
  - "[[concepts/messages]]"
  - "[[concepts/model-providers]]"
---

# 再試行ポリシー（解説）

> 原典: `raw/docs/concepts/retry.md` ・ https://docs.openclaw.ai/ja-JP/concepts/retry

## 一言まとめ

一時的な失敗に対する再試行（リトライ）の方針――**複数ステップのフロー単位ではなく HTTP リクエスト単位で再試行**し、現在のステップだけを再試行して順序を保ち、非冪等操作の重複を避ける。モデルプロバイダーとチャネル送信の両方に適用される。

## 位置づけ

[[concepts/messages]] パイプラインの「送信」と、モデル呼び出しの信頼性を支える下支え。閾値を超える待機ではモデルフェイルオーバー（[[concepts/model-providers]] 周辺）へ切り替わる点で、再試行は失敗の最初の防衛線になる。

## 仕組み・ふるまい

### デフォルト

試行回数 3、最大遅延上限 30000ms、ジッター 0.1（10%）、あとはプロバイダー既定。

### モデルプロバイダー

- 通常の短い再試行は**プロバイダー SDK に任せる**。
- Stainless ベースの SDK（Anthropic/OpenAI 等）は、再試行可能レスポンス（`408`/`409`/`429`/`5xx`）に `retry-after-ms`/`retry-after` を含むことがある。その待機が **60 秒超**なら OpenClaw は `x-should-retry: false` を注入し、SDK に即エラーを表面化させて**モデルフェイルオーバー**（別の認証プロファイル/フォールバックモデル）へ切り替えさせる。
- 上限は `OPENCLAW_SDK_RETRY_MAX_WAIT_SECONDS` で上書き（`0`/`false`/`off`/`none`/`disabled` で SDK が長い `Retry-After` を内部尊重）。

### チャネル（例: Discord / Telegram）

- レート制限（HTTP 429）・タイムアウト・5xx・一時的トランスポート障害（DNS 失敗・接続リセット・ソケットクローズ・fetch 失敗）で再試行。
- 利用可能なら各プラットフォームの `retry_after` を使い、無ければ**指数バックオフ**。Telegram の Markdown 解析エラーは再試行せずプレーンテキストにフォールバック。

## 設定・使い方の要点

- プロバイダーごとに `channels.<channel>.retry`（`attempts`/`minDelayMs`/`maxDelayMs`/`jitter`）を設定。
- 再試行はリクエストごと（メッセージ送信・メディアアップロード・リアクション・投票・ステッカー）に適用。複合フローでは**完了済みステップは再試行されない**（順序保持）。

## 注意点・落とし穴

- 60 秒超の `Retry-After` を待つより、フェイルオーバーで別経路に切り替えた方が応答性が高い（既定挙動）。SDK に長い待機を尊重させたい場合のみ上書きする。
- 非冪等操作（送信等）の重複に注意。完了済みステップを再生しない設計はこの重複を避けるため。

## 用語と略称

- **リトライ（retry）** = 失敗したリクエストの再試行
- **指数バックオフ（exponential backoff）** = 再試行間隔を指数的に伸ばす方式
- **ジッター（jitter）** = 同時再試行の集中を避ける遅延のランダム化
- **冪等（idempotent）** = 何度実行しても結果が同じであること
- **モデルフェイルオーバー** = 失敗時に別の認証プロファイル/モデルへ切り替えること

## 関連ページ

- [[concepts/retry]] — 対応する概念ページ
- [[concepts/messages]] / [[concepts/queue]]
- [[concepts/model-providers]]
