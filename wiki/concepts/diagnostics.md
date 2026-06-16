---
type: concept
aliases: [Diagnostics, 診断, doctor, troubleshooting]
tags: [diagnostics, health, doctor, troubleshooting, stability]
related:
  - "[[concepts/logging]]"
  - "[[concepts/observability]]"
  - "[[concepts/configuration]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/health]]"
  - "[[sources/gateway/doctor]]"
  - "[[sources/gateway/troubleshooting]]"
  - "[[sources/gateway/diagnostics]]"
updated: 2026-06-14
---

# 診断（Diagnostics）

OpenClaw を**運用し、不具合を直す**ための道具立て――健全性確認（health）、修復・移行（doctor）、症状起点の手順書（troubleshooting）、共有可能なサポートバンドル（診断エクスポート）。背後には**ペイロードなしの安定性レコーダー**と構造化された**診断イベント**がある。

## なぜ重要か

[[concepts/logging]]（生ログ）や [[concepts/observability]]（集計・トレース）が「見る」ための層なのに対し、診断は「**確認して・直す**」ための実務層。`openclaw status`/`health` で接続性を推測せず確認し、`openclaw doctor` が壊れた設定（[[concepts/configuration]] の厳格検証に落ちたもの）やレガシー形式・サービスドリフトを修復し、`diagnostics export` がチャット本文や認証情報を省いた**共有可能な zip** を作る。診断イベントは同時に [[concepts/observability]] のメトリクス/トレースの供給源でもある。

## 押さえる点

- **health**（[[sources/gateway/health]]）：`status`/`status --deep`/`health [--verbose|--json]`、チャネルヘルスモニター（`channelHealthCheckMinutes` 等で stale 検出→自動再起動）。
- **doctor**（[[sources/gateway/doctor]]）：`--fix`/`--repair`/`--deep`。検証失敗時の回復・レガシー正規化・サービス修復。
- **troubleshooting**（[[sources/gateway/troubleshooting]]）：症状→原因→対処のランブック。最初の一手は `status` → `doctor` → `logs`。
- **diagnostics export**（[[sources/gateway/diagnostics]]）：`gateway diagnostics export` / `/diagnostics`（1 回の exec 承認）。安定性レコーダー（`liveness.warning`/`phase.completed`）。
- 診断は既定有効（`diagnostics: { enabled: false }` で無効、バグ報告の情報が減る）。

## 関連

- [[concepts/logging]] / [[concepts/observability]] / [[concepts/configuration]]
- [[components/gateway]]
