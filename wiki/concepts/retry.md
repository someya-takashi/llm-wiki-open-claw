---
type: concept
aliases: [Retry policy, 再試行ポリシー, retry]
tags: [retry, backoff, rate-limit, failover]
related:
  - "[[concepts/messages]]"
  - "[[concepts/queue]]"
  - "[[concepts/model-providers]]"
sources:
  - "[[sources/concepts/retry]]"
updated: 2026-06-14
---

# 再試行ポリシー

**再試行ポリシー**は、一時的な失敗に対するリトライの方針――**フロー単位ではなく HTTP リクエスト単位**で再試行し、現在のステップだけを再試行して順序を保ち、非冪等操作の重複を避ける。モデルプロバイダー呼び出しとチャネル送信の両方に適用される。

## なぜ重要か

ネットワークやプロバイダーの一時障害（429・5xx・タイムアウト・接続リセット）はマルチチャネル運用で日常的に起きる。再試行は「失敗の最初の防衛線」で、**60 秒を超える `Retry-After` を待つよりモデルフェイルオーバー（別の認証プロファイル/フォールバックモデル）へ切り替える**という判断が、応答性を保つうえで効く。完了済みステップを再生しない設計が、送信などの非冪等操作の二重実行を防ぐ。

## 押さえる点

- 既定：試行 3 回・最大遅延 30000ms・ジッター 0.1。モデル側は基本 SDK 任せ、長すぎる待機は `x-should-retry: false` でフェイルオーバーへ。
- チャネル（Discord/Telegram 等）は `retry_after` 優先・無ければ指数バックオフ。設定は `channels.<ch>.retry`。
- 再試行はリクエストごと（送信・メディア・リアクション等）。

詳細は [[sources/concepts/retry]]。

## 関連

- [[concepts/messages]] / [[concepts/queue]]
- [[concepts/model-providers]]
