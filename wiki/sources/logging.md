---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/logging
source_path: raw/docs/logging.md
doc_section: root
title: "ロギング"
ingested: 2026-06-14
tags: [logging, jsonl, redaction, log-level, trace-correlation]
related:
  - "[[concepts/logging]]"
  - "[[concepts/observability]]"
  - "[[components/gateway]]"
---

# ロギング（解説）

> 原典: `raw/docs/logging.md` ・ https://docs.openclaw.ai/ja-JP/logging
>
> ℹ️ docs の**トップレベル（ルート）ドキュメント**（URL に section プレフィックス無し）。ログの**利用者向け概要**で、Gateway 内部の WS ログ/サブシステム整形は [[sources/gateway/logging]] にある。

## 一言まとめ

OpenClaw のログは **2 面**――Gateway が書く**ファイルログ（JSON lines）**と、ターミナル/Debug UI に出る**コンソール出力**。ログの保存場所・読み方・レベル/形式/秘匿化の設定を説明したページ。

## 位置づけ

[[concepts/logging]] の本体。ログ・メトリクス・トレースという 3 つの観測信号のうち「ログ」を担い、外部への送出は [[concepts/observability]]（OTLP）へつながる。Control UI/CLI でこのファイルを tail する。

## 仕組み・ふるまい

- **保存場所**：既定 `/tmp/openclaw/openclaw-YYYY-MM-DD.log`（ホストのローカル TZ、1 日 1 ファイル）。`logging.maxFileBytes`（既定 100MB）でローテーション＋番号付きアーカイブ最大 5。`logging.file` で上書き。
- **読み方**：`openclaw logs --follow`（RPC で Gateway のファイルを tail。`--local-time`/`--json`/`--plain`/`--no-color`）。Control UI の Logs タブ（`logs.tail`）。チャネル別は `openclaw channels logs --channel <ch>`。
- **JSONL レコード**：機械フィルタ用に `hostname`/`message`/`agent_id`/`session_id`/`channel` をトップレベルに持つ。会話/音声/managed-room は本文を省いた境界付きライフサイクルレコードを出す。
- **トレース相関**：有効なトレースコンテキストがあれば `traceId`/`spanId`/`parentSpanId`/`traceFlags` をトップレベルに書き、外部プロセッサが OTel span や `traceparent` 伝播と相関できる。

## 設定・使い方の要点

- 設定は `logging.*`：`level`（**ファイルログ**の JSONL レベル）・`consoleLevel`/`consoleStyle`（`pretty`/`compact`/`json`）・`file`・`redactSensitive`・`redactPatterns`。
- **`--verbose` はコンソールと WS ログの詳細度だけ**を上げ、ファイルログレベルは変えない。ファイルに詳細を残すなら `logging.level: debug|trace`。`OPENCLAW_LOG_LEVEL`/`--log-level` で一時上書き。
- **対象を絞ったモデル転送デバッグ**：`OPENCLAW_DEBUG_MODEL_TRANSPORT=1`・`OPENCLAW_DEBUG_MODEL_PAYLOAD=summary|tools|full-redacted`・`OPENCLAW_DEBUG_SSE=events|peek`・`OPENCLAW_DEBUG_CODE_MODE=1`（全ログを `debug` に上げずに済む）。
- **モデル呼び出しのサイズ/タイミング**：`requestPayloadBytes`/`responseStreamBytes`/`timeToFirstByteMs`/`durationMs`（生のプロンプト/レスポンスは取らない）。

## 注意点・落とし穴

- **秘匿化（redaction）**：`logging.redactSensitive`（`off`/`tools`〔既定〕）＋`redactPatterns`（正規表現）。一致は先頭 6＋末尾 4 文字を残してマスク。⚠️ **安全境界サーフェス**（Control UI ツールイベント・`sessions_history` 出力・診断エクスポート・provider error・exec 承認表示・WS プロトコルログ）は `redactSensitive: "off"` でも常に秘匿化される。
- ログが空→Gateway が動いていて `logging.file` に書いているか確認。Gateway 不達→まず `openclaw doctor`。

## 用語と略称

- **JSONL / JSON lines** = 1 行 1 JSON オブジェクトのログ形式
- **tail** = ログ末尾を追って表示すること
- **TTY** = 端末（color/整形の判定に使う）
- **SSE** = Server-Sent Events（ストリーミングのイベント形式）
- **redaction（秘匿化）** = 機密値をマスクすること
- **traceId / span** = 分散トレースの識別子（→ [[concepts/observability]]）

## 関連ページ

- [[concepts/logging]] — 対応する概念ページ
- [[sources/gateway/logging]] — Gateway 内部の WS ログ/サブシステム整形
- [[concepts/observability]] / [[concepts/diagnostics]] / [[components/gateway]]
