---
type: concept
aliases: [Observability, 可観測性, telemetry, テレメトリ]
tags: [observability, metrics, traces, prometheus, opentelemetry, otlp]
related:
  - "[[concepts/logging]]"
  - "[[concepts/diagnostics]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/prometheus]]"
  - "[[sources/gateway/opentelemetry]]"
updated: 2026-06-14
---

# 可観測性（Observability）

OpenClaw は内部の**診断イベント**（モデル実行・メッセージフロー・セッション・キュー・exec の構造化レコード）を、**メトリクス・トレース・ログ**として外部の監視基盤へ送れる。2 つの公式 Plugin――**Prometheus**（プル型・メトリクスのみ）と **OpenTelemetry**（プッシュ型・3 信号すべて、OTLP/HTTP）――がそのサーフェス。

## なぜ重要か

「いま正しく・速く動いているか」を本番で継続監視し、トークン使用量・コスト・レイテンシ・キュー深さ・セッション回復などを可視化するための層。ログ（[[concepts/logging]]）が「個々の事象」を、可観測性が「集計とトレンド・分散トレース」を担う。**診断イベントが共通の供給源**で、同じイベントが Prometheus メトリクスにも OTel span/ログにもなる。`gen_ai_client_token_usage` のように OTel GenAI 規約に従う指標は、他の GenAI サービスと横断比較できる。

## 押さえる点

- **Prometheus**（[[sources/gateway/prometheus]]）：`GET /api/diagnostics/prometheus` を**スクレイプ**（プル）。**Gateway 認証必須**（公開 `/metrics` にしない）。時系列は 2048 系列上限、`series_dropped_total` が高カーディナリティ漏れの警報。
- **OpenTelemetry**（[[sources/gateway/opentelemetry]]）：OTLP/HTTP で任意のコレクター（Grafana/Datadog/Honeycomb 等）へ**プッシュ**。メトリクス＋トレース（`openclaw.run`/`model.call` 等）＋ログ。
- どちらも `diagnostics: { enabled: true }` が前提。生のプロンプト/応答・セッション ID・秘密値は**出力に現れない**（境界付き・低カーディナリティ）。両方欲しいなら OTel Collector で Prometheus にブリッジ。

## 関連

- [[concepts/logging]] — ログ信号と trace 相関
- [[concepts/diagnostics]] — 診断イベント/サポートバンドルの供給源
- [[components/gateway]]
