---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/opentelemetry
source_path: raw/docs/gateway/opentelemetry.md
doc_section: gateway
title: "OpenTelemetry エクスポート"
ingested: 2026-06-14
tags: [opentelemetry, otlp, traces, metrics, logs, observability, plugin]
related:
  - "[[concepts/observability]]"
  - "[[concepts/diagnostics]]"
  - "[[components/gateway]]"
---

# OpenTelemetry エクスポート（解説）

> 原典: `raw/docs/gateway/opentelemetry.md` ・ https://docs.openclaw.ai/ja-JP/gateway/opentelemetry

## 一言まとめ

公式 `diagnostics-otel` Plugin で、診断情報を **OTLP/HTTP（protobuf）** により **メトリクス・トレース・ログ**として OpenTelemetry 互換コレクター/バックエンド（Grafana/Datadog/Honeycomb/Tempo 等）へ**プッシュ**する。

## 位置づけ

[[concepts/observability]] の「テレメトリ・プッシュ」面（3 信号すべて）。[[concepts/diagnostics]] の診断イベントを購読して OTLP 化する。メトリクスのみのプル型は [[sources/gateway/prometheus]]、ローカルファイルログは [[sources/logging]]。

## 仕組み・ふるまい

- **全体**：診断イベント（モデル実行・メッセージフロー・セッション・キュー・exec の構造化プロセス内レコード）→ `diagnostics-otel` Plugin が購読 → OTLP/HTTP で metrics/traces/logs としてエクスポート。
- **エクスポートされる信号**：
  - **メトリクス**：モデル使用量（トークン/コスト、GenAI 規約の `gen_ai_client_token_usage`）・メッセージフロー・Talk・キュー/セッション・セッション生存状態・ハーネスライフサイクル・実行・診断内部（メモリ/ツールループ）。
  - **トレース（span）**：`openclaw.run`/`openclaw.harness.run`/`openclaw.model.call`/`openclaw.context.assembled`/`openclaw.message.delivery` 等（QA の OTel スモークで検証される形）。
  - **ログ**：ファイルログと同じ境界付き属性で OTLP ログへ。
- **トレース相関**：ログの `traceId`/`spanId` と span、信頼プロバイダーの `traceparent` 伝播が `traceId` で結合できる（→ [[sources/logging]]）。

## 設定・使い方の要点

- 導入：`diagnostics-otel` Plugin を有効化＋`diagnostics.enabled: true`。OTLP エンドポイント・ヘッダー・サンプリング/フラッシュは設定リファレンス／環境変数で指定。
- **プライバシーとコンテンツキャプチャ**：既定は内容を取らない（メトリクス/span は境界付き属性のみ）。コンテンツキャプチャは明示オプトイン。
- Prometheus と両方欲しい場合は、OTel Collector の `prometheus`/`prometheusremotewrite` エクスポーターでブリッジ。
- エクスポーター無しでも診断イベントはプロセス内で出る（Plugin/カスタム sink 用に `diagnostics: { enabled: true }`）。

## 注意点・落とし穴

- **OTLP** はオープン標準なので、受け側がコード変更なしで使える反面、コレクターのエンドポイント/認証設定を誤ると送れない。
- メトリクスのみで十分なら Prometheus（プル）の方が運用が軽いことも（[[sources/gateway/prometheus]] の比較）。

## 用語と略称

- **OpenTelemetry（OTel）** = メトリクス/トレース/ログの観測標準
- **OTLP** = OpenTelemetry Protocol（テレメトリ送信プロトコル）
- **span / trace** = 処理 1 区間 / それらを束ねた一連の処理
- **GenAI semantic conventions** = 生成 AI 向けの OTel 標準属性
- **プッシュ vs プル** = 送信側が送る / 受信側が取りに来る

## 関連ページ

- [[concepts/observability]] — 対応する概念ページ
- [[sources/gateway/prometheus]] — メトリクスのみのプル型
- [[sources/logging]] / [[sources/gateway/diagnostics]] / [[components/gateway]]
