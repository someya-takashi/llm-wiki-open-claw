---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/troubleshooting
source_path: raw/docs/gateway/troubleshooting.md
doc_section: gateway
title: "Gateway トラブルシューティング"
ingested: 2026-06-14
tags: [troubleshooting, runbook, diagnostics, gateway, channels]
related:
  - "[[concepts/diagnostics]]"
  - "[[components/gateway]]"
  - "[[sources/gateway/doctor]]"
---

# Gateway トラブルシューティング（解説）

> 原典: `raw/docs/gateway/troubleshooting.md` ・ https://docs.openclaw.ai/ja-JP/gateway/troubleshooting
>
> ℹ️ これは**詳細なランブック**。素早いトリアージは help/troubleshooting から。本解説は症状の地図として要点を示す（個別手順は原典 zip/コマンドを参照）。

## 一言まとめ

Gateway の不具合を**症状起点で診断・復旧**するための詳細手順書。「まず何を実行するか」と、よくある症状ごとの原因・対処を並べる。

## 位置づけ

[[concepts/diagnostics]] の「症状→復旧」面。最初の一手は [[sources/gateway/health]]（状態確認）と [[sources/gateway/doctor]]（修復）、深掘りは [[sources/gateway/diagnostics]]（バンドル）・[[sources/logging]]（ログ）。

## 仕組み・ふるまい（症状の地図）

まず順に `openclaw status` → `openclaw doctor` → `openclaw logs --follow`。主な症状カテゴリ：

- **設定/インストール**：分断されたインストールと新設定ガード、Skill シンボリックリンクがパスエスケープでスキップ、`Gateway が無効な設定を拒否`（→ [[sources/gateway/doctor]] で修復）。
- **モデル/プロバイダー**：Anthropic 429（長いコンテキストは追加利用が必要）、ローカル OpenAI 互換は直接プローブを通るがエージェント実行が失敗（→ ツールサーフェス縮小・[[sources/concepts/experimental-features]] のリーンモード）。
- **接続/UI**：Gateway サービスが動いていない、Gateway probe 警告（複数到達）、ダッシュボード Control UI の接続性、**認証詳細コードのクイックマップ**。
- **メッセージング**：返信がない、チャネルは接続済みだがメッセージが流れない、Cron と Heartbeat の配送、Node はペアリング済みだがツールが失敗、ブラウザツールが失敗。
- **アップグレード後の突然の不具合**。

## 設定・使い方の要点

- 認証エラーは「認証詳細コードのクイックマップ」で接続認証（[[concepts/authentication]]）/モデル認証（[[sources/gateway/authentication]]）を切り分ける。
- メッセージが流れない→許可リスト/メンションゲート（[[sources/gateway/config-channels]]）と [[sources/gateway/health]] のチャネルプローブ。
- バグ報告は `openclaw gateway diagnostics export` の zip を添付（→ [[sources/gateway/diagnostics]]）。

## 注意点・落とし穴

- `Gateway が無効な設定を拒否した` 場合、診断コマンド（doctor/logs/health/status）だけが動く（[[concepts/configuration]]）。
- 「セッション行が無い＝切断」ではない（保存状態の読み取り。ライブはヘルスで確認）。

## 用語と略称

- **ランブック（runbook）** = 症状起点の診断・復旧手順集
- **トリアージ** = 問題の切り分け・優先度付け
- **probe 警告** = 複数 Gateway 到達などの能動確認での警告
- **EADDRINUSE** = ポート使用中エラー

## 関連ページ

- [[concepts/diagnostics]] — 対応する概念ページ
- [[sources/gateway/doctor]] / [[sources/gateway/health]] / [[sources/gateway/diagnostics]] / [[sources/logging]]
- [[concepts/configuration]] / [[concepts/authentication]] / [[components/gateway]]
