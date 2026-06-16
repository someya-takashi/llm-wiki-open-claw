---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/doctor
source_path: raw/docs/gateway/doctor.md
doc_section: gateway
title: "Doctor（修復・移行）"
ingested: 2026-06-14
tags: [doctor, repair, migration, diagnostics, config]
related:
  - "[[concepts/diagnostics]]"
  - "[[concepts/configuration]]"
  - "[[components/gateway]]"
---

# Doctor（修復・移行）（解説）

> 原典: `raw/docs/gateway/doctor.md` ・ https://docs.openclaw.ai/ja-JP/gateway/doctor

## 一言まとめ

`openclaw doctor` は OpenClaw の**修復＋移行ツール**――古くなった設定/状態を直し、健全性を確認し、実行可能な修復手順を提示する。検証に失敗した設定の復旧や、レガシー形式の正規化の主役。

## 位置づけ

[[concepts/diagnostics]] の「修復・移行」面。[[concepts/configuration]] の厳格検証に落ちたときの回復経路（最後に正常だったコピーへの復元・プレフィックス付き設定の修復）であり、サービスドリフトの監査・修復も行う。

## 仕組み・ふるまい

- **モード**：`openclaw doctor`（診断のみ）／`--yes`（対話なしで安全な修復を適用）／`--repair`／`--repair --force`／`--non-interactive`（ヘッドレス/自動化）／`--deep`（システムレベルのサービススキャンを追加）。
- **実行内容（概要）**：設定の検証と修復（不明キー/壊れた設定・プレフィックス付き設定・最後に正常だったコピーの復元）、レガシー認証ストレージの正規化（フラットな `auth-profiles.json` → 正規形、`type: "aws-sdk"` の移動）、サービス設定のドリフト監査/修復（launchd/systemd/schtasks。同一プロファイル/ポートへの二重インストール拒否）、各種状態の移行。
- **Dreams UI の backfill/reset**：[[concepts/dreaming]] の根拠付きバックフィルやリセットのフローも doctor 系から扱える。
- 致命的でない検出と「実行可能な次の一手」を提示するのが基本姿勢。

## 設定・使い方の要点

- 設定が起動を拒否されたら：`openclaw doctor` で原因確認 → `openclaw doctor --fix`（または `--yes`）で修復（→ [[concepts/configuration]] の検証失敗フロー）。
- サービスのドリフト：`doctor` がランチャー設定を監査・修復。システムユニットがライフサイクルを所有する場合は `OPENCLAW_SERVICE_REPAIR_POLICY=external` で doctor のユーザーレベル自動インストールを抑制。

## 注意点・落とし穴

- `--repair --force` は強い修復。意図を理解して使う。
- doctor はレガシー形式を移すとき**元ファイルの横にバックアップ**（例 `.legacy-flat.*.bak`）を残す。
- 検証失敗時に Gateway は「最後に正常だったコピー」を保持するが**自動復元はしない**（doctor で明示的に）。

## 用語と略称

- **doctor** = 設定/状態の修復・移行・健全性チェックを行う CLI
- **ドリフト（drift）** = 期待と実際のサービス設定のずれ
- **`--fix` / `--repair`** = 検出した問題を修復する実行モード
- **launchd / systemd / schtasks** = OS のサービス監視機構

## 関連ページ

- [[concepts/diagnostics]] — 対応する概念ページ
- [[concepts/configuration]] — 検証失敗時の修復フロー
- [[sources/gateway/health]] / [[sources/gateway/troubleshooting]] / [[components/gateway]]
