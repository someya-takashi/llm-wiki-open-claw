---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/qa-matrix
source_path: raw/docs/concepts/qa-matrix.md
doc_section: concepts
title: "Matrix QA"
ingested: 2026-06-14
tags: [qa, matrix, testing, e2ee, live-transport, docker]
related:
  - "[[concepts/qa-automation]]"
  - "[[sources/concepts/qa-e2e-automation]]"
  - "[[channels/matrix]]"
---

# Matrix QA（解説）

> 原典: `raw/docs/concepts/qa-matrix.md` ・ https://docs.openclaw.ai/ja-JP/concepts/qa-matrix
>
> ℹ️ **メンテナー専用ツール**。パッケージ済みリリースは `qa-lab` を省くため、`openclaw qa` はソースチェックアウトからのみ動く。

## 一言まとめ

Matrix QA レーンは、Docker 内の**使い捨て Tuwunel ホームサーバー**に対してバンドルの `@openclaw/matrix` プラグインを実行し、Matrix（分散型チャットプロトコル）の振る舞いを**実トランスポートでライブ検証**する。一時的な driver/SUT/observer アカウントと事前投入ルームを使う。

## 位置づけ

[[concepts/qa-automation]] のライブトランスポートレーンの 1 つで、シナリオ数と Docker ベースのホームサーバー構築のため専用ページを持つ。検証対象は [[channels/matrix]] のチャネル Plugin。ユニットテストでは届かないトランスポート挙動（mention gating・allowlist・スレッド・再起動 replay・**E2EE**＝End-to-End Encryption のブートストラップ/復旧/検証）を実証する。

## 仕組み・ふるまい

### レーンの動作

1. Docker 内に使い捨て Tuwunel ホームサーバーを構築（既定イメージ `tuwunel:v1.5.1`、サーバー名 `matrix-qa.test`、ポート `28008`）。
2. 一時ユーザー 3 つを登録：`driver`（受信トラフィックを送る）・`sut`（テスト対象の OpenClaw Matrix アカウント）・`observer`（第三者トラフィックを観測）。
3. シナリオに必要なルームを事前投入（main・threading・media・restart・E2EE・verification DM 等）。
4. SUT にスコープした実 Matrix プラグインで子 OpenClaw Gateway を起動（子では `qa-channel` を読み込まない）。
5. シナリオを順に実行し、driver/observer クライアントでイベントを観測。
6. ホームサーバーを破棄し、レポートとサマリーを書き出して終了。

### プロファイルとシナリオ

`--profile` で実行範囲を選ぶ：`all`（既定・網羅）/ `fast`（リリースゲート用サブセット）/ `transport` / `media` / `e2ee-smoke` / `e2ee-deep` / `e2ee-cli`。シナリオはスレッド・DM/ルーム・ストリーミング・メディア・ルーティング・リアクション・承認・再起動/replay・mention/allowlist・E2EE・E2EE CLI などのカテゴリに分かれる（正確なカタログは `extensions/qa-matrix/.../scenario-catalog.ts`）。

## 設定・使い方の要点

- 実行：`pnpm openclaw qa matrix --profile fast --fail-fast`（リリースゲート）。無印は `--profile all` で最初の失敗でも止まらない。`--scenario <id>` で個別実行。
- モデルプロバイダーは設定可能：`--provider-mode mock-openai`（決定的モック）/ `live-frontier`（既定・ライブ）。Matrix QA は使い捨てユーザーをローカル生成するため `--credential-source`/`--credential-role` は受け付けない。
- 出力：`matrix-qa-report.md`・`matrix-qa-summary.json`・`matrix-qa-observed-events.json`・`matrix-qa-output.log` を `.artifacts/qa-e2e/matrix-<timestamp>/` に。本文は既定で伏せ字化（`OPENCLAW_QA_MATRIX_CAPTURE_CONTENT=1` で保持、機微扱い）。

## 注意点・落とし穴

- 実行が終盤でハングする：`matrix-js-sdk` のネイティブ暗号ハンドルがハーネスより長く生存することがあり、既定はアーティファクト書き込み後に `process.exit` を強制する（`OPENCLAW_QA_MATRIX_DISABLE_FORCE_EXIT=1` を解除しているとプロセスが残る）。
- クリーンアップエラー：出力された `docker compose ... down --remove-orphans` を手動実行してポートを解放。

## 用語と略称

- **Matrix** = 分散型のオープンチャットプロトコル
- **Tuwunel** = テストに使う Matrix ホームサーバー実装
- **E2EE** = End-to-End Encryption（端末間暗号化）
- **SUT** = System Under Test（テスト対象の OpenClaw ボット）
- **homeserver** = Matrix のアカウント/ルームをホストするサーバー

## 関連ページ

- [[concepts/qa-automation]] — QA スタック全体とライブトランスポート契約
- [[sources/concepts/qa-e2e-automation]] — QA 概要
- [[channels/matrix]]（未作成）
