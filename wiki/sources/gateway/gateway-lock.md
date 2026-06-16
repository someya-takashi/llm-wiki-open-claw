---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/gateway-lock
source_path: raw/docs/gateway/gateway-lock.md
doc_section: gateway
title: "Gateway ロック"
ingested: 2026-06-14
tags: [gateway, lock, port, single-instance, eaddrinuse]
related:
  - "[[components/gateway]]"
  - "[[concepts/architecture]]"
  - "[[sources/gateway/multiple-gateways]]"
---

# Gateway ロック（解説）

> 原典: `raw/docs/gateway/gateway-lock.md` ・ https://docs.openclaw.ai/ja-JP/gateway/gateway-lock

## 一言まとめ

**同じホスト・同じベースポートでは Gateway を 1 つだけ**走らせ、ポート競合では明確なエラーで即失敗、クラッシュ後も古いロックを残さず復旧する――という Gateway の起動ロック機構。

## 位置づけ

[[components/gateway]] の不変条件「ホストごと 1 つ」を実装で強制する仕組み。複数インスタンスを意図的に動かす方法は [[sources/gateway/multiple-gateways]]、`EADDRINUSE` 等の診断は [[sources/gateway/troubleshooting]]。

## 仕組み・ふるまい

- 起動時、状態ロックディレクトリ配下の**設定ごとのロックファイル**を取得し、設定ポートに既存リスナーがあるか調べる。ロック所有者が居ない/ポートが空き/ロックが古い場合は再取得して続行。
- 次に**排他的 TCP リスナー**で HTTP/WebSocket（既定 `ws://127.0.0.1:18789`）をバインド。
- バインドが `EADDRINUSE` で失敗すると `GatewayLockError("another gateway instance is already listening on ws://127.0.0.1:<port>")` を投げる。その他のバインド失敗は `GatewayLockError("failed to bind gateway socket ...")`。
- シャットダウン時に HTTP/WS サーバーを閉じてロックファイルを削除。

## 設定・使い方の要点

- ポートが**別プロセス**に占有されていても同じエラーになる。ポートを空けるか `openclaw gateway --port <port>` で別ポートを選ぶ。
- **サービススーパーバイザー下**：正常な `/healthz` 応答元を検出した新プロセスは制御を譲る。systemd では重複起動元がコード 78 で終了し、既定の `RestartPreventExitStatus=78` で `Restart=always` のループを防ぐ。
- macOS アプリは Gateway 生成前に軽量な PID ガードも持つ（ランタイムロックはロックファイル＋HTTP/WS バインドで強制）。

## 注意点・落とし穴

- 「別の Gateway が listening」エラーは、**意図しない二重起動**か**他プロセスのポート占有**のどちらでも出る。意図的な複数 Gateway なら一意ポート＋プロファイルにする（[[sources/gateway/multiple-gateways]]）。

## 用語と略称

- **ロックファイル** = 1 ホスト 1 Gateway を保証する排他制御ファイル
- **EADDRINUSE** = ポート使用中の OS エラー
- **`/healthz`** = サービス監視用のヘルスエンドポイント
- **PID ガード** = プロセス ID による二重起動防止

## 関連ページ

- [[components/gateway]] / [[concepts/architecture]]
- [[sources/gateway/multiple-gateways]] / [[sources/gateway/troubleshooting]]
