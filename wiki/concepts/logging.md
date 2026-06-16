---
type: concept
aliases: [Logging, ロギング, logs]
tags: [logging, jsonl, redaction, observability]
related:
  - "[[concepts/observability]]"
  - "[[concepts/diagnostics]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/logging]]"
  - "[[sources/gateway/logging]]"
updated: 2026-06-14
---

# ロギング

OpenClaw のログは **2 面**――[[components/gateway]] が書く**ファイルログ（JSON lines）**と、ターミナル/Debug UI に出る**コンソール出力**。観測の 3 信号（ログ・メトリクス・トレース）のうち「ログ」を担い、`openclaw logs --follow` や Control UI の Logs タブで読む。

## なぜ重要か

何が起きたかを後から追える**一次情報**。OpenClaw のログ設計のポイントは、**ファイルログとコンソールのレベルを分離**（`--verbose` はコンソール/WS のみで、ファイルレベルは上げない）し、**秘匿化（redaction）を全シンクに適用**すること――一致するシークレットはディスクに書かれる前にマスクされ、Control UI ツールイベント・`sessions_history`・診断エクスポート等の**安全境界サーフェスは `redactSensitive: "off"` でも常に秘匿化**される。さらに `traceId`/`spanId` をトップレベルに書くことで [[concepts/observability]] の span と相関できる。

## 押さえる点

- 設定 `logging.*`：`level`（ファイル JSONL）・`consoleLevel`/`consoleStyle`・`file`・`redactSensitive`（`off`/`tools`）・`redactPatterns`。`OPENCLAW_LOG_LEVEL`/`--log-level` で一時上書き。
- 既定 `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（100MB ローテーション）。JSONL に `agent_id`/`session_id`/`channel` 等のフィルタ用フィールド。
- **対象を絞ったデバッグ**：`OPENCLAW_DEBUG_MODEL_TRANSPORT`/`_PAYLOAD`/`OPENCLAW_DEBUG_SSE` で全ログを上げずにモデル転送だけ詳細化。
- Gateway 内部の WS ログ/サブシステム整形は [[sources/gateway/logging]]、利用者向け概要は [[sources/logging]]。

## 関連

- [[concepts/observability]] — メトリクス/トレース/ログの外部エクスポート
- [[concepts/diagnostics]] / [[components/gateway]]
