---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/prometheus
source_path: raw/docs/gateway/prometheus.md
doc_section: gateway
title: "Prometheus メトリクス"
ingested: 2026-06-14
tags: [prometheus, metrics, observability, plugin, scrape, cardinality]
related:
  - "[[concepts/observability]]"
  - "[[components/gateway]]"
  - "[[concepts/diagnostics]]"
---

# Prometheus メトリクス（解説）

> 原典: `raw/docs/gateway/prometheus.md` ・ https://docs.openclaw.ai/ja-JP/gateway/prometheus

## 一言まとめ

公式 `diagnostics-prometheus` Plugin で、内部診断イベントを **Prometheus テキストエンドポイント** `GET /api/diagnostics/prometheus` として公開する（**プル型**）。メトリクスのみ（トレース/ログは OpenTelemetry 側）。

## 位置づけ

[[concepts/observability]] の「メトリクス（プル）」面。[[concepts/diagnostics]] の診断イベントを購読してメトリクス化し、Prometheus+Grafana スタックに供給する。トレース/ログまで欲しいなら [[sources/gateway/opentelemetry]]（プッシュ型）。

## 仕組み・ふるまい

- エンドポイントは標準 exposition format（`text/plain; version=0.0.4`）。⚠️ **Gateway 認証（operator スコープ）が必要**で、公開の未認証 `/metrics` として晒さない（他の operator API と同じ認証でスクレイプ）。
- **主なメトリクス**：`openclaw_run_completed_total`/`_duration_seconds`（実行）、`openclaw_model_call_total`/`_duration_seconds`、`openclaw_model_tokens_total`・`openclaw_gen_ai_client_token_usage`（OTel GenAI 規約）・`openclaw_model_cost_usd_total`、`openclaw_tool_execution_*`、`openclaw_harness_run_*`、`openclaw_message_processed_*`/`_delivery_*`、`openclaw_talk_*`、`openclaw_queue_lane_size`/`_wait_seconds`、`openclaw_session_state_total`/`_queue_depth`/`_recovery_*`、`openclaw_memory_*`、`openclaw_telemetry_exporter_total`、`openclaw_prometheus_series_dropped_total`。
- **ラベルポリシー**：境界付き・低カーディナリティ。`runId`/`sessionKey`/`sessionId`/`callId` 等の生識別子は出さず、規約外の値は `unknown`/`other`/`none` に。時系列はメモリ内 **2048 系列**上限で、超過分は破棄して `openclaw_prometheus_series_dropped_total` を増やす（＝高カーディナリティ漏れの強いシグナル）。
- プロンプト/応答テキスト・Talk 文字起こし/音声/通話 ID・生プロバイダーリクエスト ID・セッションキー/ID・ホスト名/パス/秘密値は**出力に決して現れない**。

## 設定・使い方の要点

- 導入：`openclaw plugins install clawhub:@openclaw/diagnostics-prometheus` → `plugins.allow`/`entries` で有効化＋`diagnostics: { enabled: true }` → **Gateway 再起動**（HTTP ルートは Plugin 起動時に登録）。
- スクレイプ：`curl -H "Authorization: Bearer $OPENCLAW_GATEWAY_TOKEN" http://127.0.0.1:18789/api/diagnostics/prometheus`。Prometheus 側は `authorization.credentials_file` でトークンを渡す。
- PromQL 例：`sum by (provider)(rate(openclaw_model_tokens_total[1m]))`、`histogram_quantile(0.95, sum by (le,provider,model)(rate(openclaw_run_duration_seconds_bucket[5m])))`。
- **プロバイダー横断ダッシュボードは `gen_ai_client_token_usage` を優先**（OTel GenAI 規約準拠）。

## 注意点・落とし穴

- 空レスポンス→`diagnostics.enabled: true`・Plugin が `plugins list --enabled` に出る・トラフィックがある（イベント発生後にだけ行が出る）。`401`→operator スコープのトークン/パスワード。
- `series_dropped_total` が増える→2048 上限超過＝高カーディナリティの原因を直す（上限は自動で上げない）。再起動でカウンターは 0 リセット → PromQL は `rate()`/`increase()` を使う。

## 用語と略称

- **Prometheus** = プル型のメトリクス監視システム
- **スクレイプ（scrape）** = Prometheus が定期的にメトリクスを取得すること
- **カーディナリティ** = ラベル値の種類数（多いと時系列が爆発する）
- **PromQL** = Prometheus のクエリ言語
- **SLO** = Service Level Objective（サービス品質目標）
- **exposition format** = Prometheus のメトリクステキスト形式

## 関連ページ

- [[concepts/observability]] — 対応する概念ページ
- [[sources/gateway/opentelemetry]] — プッシュ型（メトリクス＋トレース＋ログ）
- [[sources/gateway/diagnostics]] / [[components/gateway]]
