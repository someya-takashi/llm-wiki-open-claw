---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/diagnostics
source_path: raw/docs/gateway/diagnostics.md
doc_section: gateway
title: "診断情報のエクスポート"
ingested: 2026-06-14
tags: [diagnostics, export, bug-report, stability, privacy, redaction]
related:
  - "[[concepts/diagnostics]]"
  - "[[components/gateway]]"
  - "[[concepts/observability]]"
---

# 診断情報のエクスポート（解説）

> 原典: `raw/docs/gateway/diagnostics.md` ・ https://docs.openclaw.ai/ja-JP/gateway/diagnostics

## 一言まとめ

バグ報告用に、**サニタイズ済みの Gateway 状態・ヘルス・ログ・設定形状・安定性イベントをまとめたローカル zip** を作る仕組み。共有可能になるよう設計され、認証情報やチャット本文は省略/マスクされる。

## 位置づけ

[[concepts/diagnostics]] の「サポートバンドル」面。ヘルス（[[sources/gateway/health]]）や doctor（[[sources/gateway/doctor]]）と並ぶ運用診断で、外部コレクターへ流す版が [[concepts/observability]]（OpenTelemetry）。

## 仕組み・ふるまい

- **作成**：`openclaw gateway diagnostics export [--output <zip>] [--json]`。zip に `summary.md`（人間向け）/`diagnostics.json`（機械可読）/`manifest.json`/サニタイズ済み設定形状/ログ要約/Gateway 状態・ヘルススナップショット/`stability/latest.json`。Gateway が異常でもローカルログ・設定形状・最新安定性バンドルは収集される。
- **チャットコマンド `/diagnostics [note]`**（所有者）：診断前文＋**1 回の exec 承認**（allow-all で承認しない）→ バンドルパス・要約・プライバシーノート・関連セッション ID を返信。グループでは詳細を共有チャットに出さず所有者へ非公開送信。Codex ハーネス使用時は同じ承認で OpenAI へのフィードバックアップロードも対象（拒否すれば送信しない）。
- **安定性レコーダー**：診断有効時、既定で**ペイロードなしの境界付き安定性ストリーム**を記録（運用上の事実のみ）。`diagnostic.liveness.warning`（イベントループ遅延/使用率・CPU コア比・active/waiting/queued セッション数）、`diagnostic.phase.completed`（起動フェーズのタイミング）。確認は `openclaw gateway stability [--type ...|--json|--bundle latest [--export]]`。致命的終了/再起動失敗時のバンドルは `~/.openclaw/logs/stability/`。

## 設定・使い方の要点

- オプション：`--log-lines`/`--log-bytes`（取り込み量）・`--url`/`--token`/`--password`/`--timeout`（状態取得）・`--no-stability-bundle`。
- 診断は既定で有効。無効化は `diagnostics: { enabled: false }`（バグ報告の情報が減るが通常ログには影響しない）。

## 注意点・落とし穴

- **プライバシーモデル**：保持＝サブシステム/Plugin/プロバイダー/チャネル ID・モード・ステータスコード・所要時間/バイト数・キュー/メモリ読み・サニタイズ済みログメタ。省略/マスク＝**チャット本文・プロンプト・webhook 本文・ツール出力・認証情報/トークン/Cookie/秘密値・生のリクエスト/レスポンス本文・アカウント/メッセージ/生セッション ID・ホスト名/ユーザー名**。それでも**確認するまではシークレット扱い**。
- `/diagnostics` の承認は allow-all で通さない（1 回の明示承認に限定）。

## 用語と略称

- **診断バンドル / サポートバンドル** = 共有可能な診断 zip
- **安定性レコーダー** = ペイロードなしで運用イベントを記録する仕組み
- **liveness warning** = プロセスは生きているが飽和しているサイン
- **RSS** = Resident Set Size（メモリ常駐量）
- **サニタイズ / リダクト** = 機密を除去/マスクすること

## 関連ページ

- [[concepts/diagnostics]] — 対応する概念ページ
- [[sources/gateway/health]] / [[sources/gateway/doctor]] / [[sources/gateway/troubleshooting]]
- [[concepts/observability]] / [[components/gateway]]
