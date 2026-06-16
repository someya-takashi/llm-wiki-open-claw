---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/qa-e2e-automation
source_path: raw/docs/concepts/qa-e2e-automation.md
doc_section: concepts
title: "QA の概要"
ingested: 2026-06-14
tags: [qa, testing, e2e, live-transport, maintainer, mantis]
related:
  - "[[concepts/qa-automation]]"
  - "[[sources/concepts/qa-matrix]]"
  - "[[channels/slack]]"
---

# QA の概要（解説）

> 原典: `raw/docs/concepts/qa-e2e-automation.md` ・ https://docs.openclaw.ai/ja-JP/concepts/qa-e2e-automation
>
> ℹ️ これは **メンテナー（OpenClaw 開発者）向け**の社内 QA スタックの解説。パッケージ済みリリースには QA Lab が含まれず、`openclaw qa` はソースチェックアウトからのみ動く。OpenClaw を「使う」だけのユーザーには直接は関係しないが、製品がどう検証されているかを知る手がかりになる。

## 一言まとめ

プライベート QA スタックは、単一のユニットテストより現実的に・**実際のチャネルの形に近い方法**で OpenClaw を検証するための仕組み。合成チャネル・デバッガー UI・ライブトランスポートアダプターから成り、`pnpm openclaw qa <subcommand>` で実行する。

## 位置づけ

OpenClaw 本体（[[components/gateway]] とチャネル統合）が「実際のメッセージング上で正しく振る舞うか」を回帰検証する層。個別チャネル（[[channels/slack]] など）や [[concepts/agent-loop]] の挙動を、ユニットテストでは届かないトランスポート実体で確かめる。Matrix の詳細は別ページ [[sources/concepts/qa-matrix]]。

## 仕組み・ふるまい

### 構成要素

- `qa-channel`：DM・チャネル・スレッド・リアクション・編集・削除を備えた**合成メッセージチャネル**（広範なプロダクト挙動スイート）。
- `qa-lab`：トランスクリプト観察・受信メッセージ注入・Markdown レポート出力を行う**デバッガー UI と QA バス**。汎用シナリオ実行・ワーカー同時実行・アーティファクト書き込み・レポート生成という「共有ホスト」を所有。
- `qa-matrix` ほかの runner plugin：子 QA Gateway 内で**実チャネルを駆動するライブトランスポートアダプター**。
- `qa/`：リポジトリ管理のシードアセット（シナリオ Markdown）。
- [[concepts/mantis]]（未作成）：実トランスポート・ブラウザのスクリーンショット・VM 状態・PR 証拠を要するバグの、修正前後のライブ検証。

### コマンドサーフェス（抜粋）

`qa run`（セルフチェック）・`qa suite`（リポジトリ管理シナリオを QA Gateway レーンで実行。`--runner multipass` で使い捨て Linux VM）・`qa coverage`（シナリオ網羅インベントリ）・`qa character-eval`（複数ライブモデルで人格 QA、判定付きレポート）・`qa ui`/`qa up`（QA Lab UI）・`qa matrix|telegram|discord|slack`（ライブトランスポートレーン）・`qa mantis`（修正前後検証）。

### ライブトランスポート契約

Matrix・Telegram・Discord・Slack の各ライブレーンは、独自形状を発明せず**1 つの契約チェックリスト**を共有する（canary・mention gating・bot-to-bot・allowlist ブロック・トップレベル返信・再起動レジューム・スレッドフォローアップ/分離・リアクション観測 等）。`qa-channel` は広範な合成スイートとして別に維持され、この契約マトリクスには意図的に含めない。各レーンは 2 ボット（driver＋SUT）で実チャネルを対象に動き、Markdown レポート/JSON サマリー/観測イベントを `.artifacts/qa-e2e/...` に書き出す。

## 設定・使い方の要点

- 2 ペインの QA サイト（左＝Control UI ダッシュボード、右＝QA Lab）を `pnpm qa:lab:up` で起動。`qa:lab:up:fast`＋`qa:lab:watch` で UI を高速反復。
- OpenTelemetry トレーススモークは `pnpm qa:otel:smoke`（`openclaw.run`/`harness.run`/`model.call`/`context.assembled`/`message.delivery` のスパン形状を検証）。
- プロバイダーモック：`mock-openai`（決定的・パリティゲート既定）と `aimock`（実験プロトコル/記録再生/カオス）。認証情報は環境変数または共有 **Convex プール**（`--credential-source convex`、排他リース）。

## 注意点・落とし穴

- **ソースチェックアウト専用**：npm tarball は QA Lab を省くため、パッケージ Docker リリースレーンでは `qa` を実行しない。診断計装を変えたらビルド済みソースから `pnpm qa:otel:smoke` を使う。
- 観測アーティファクトのメッセージ本文は既定で伏せ字化（`*_CAPTURE_CONTENT=1` で保持できるが機微情報として扱う）。
- 新チャネル追加は「トランスポートアダプター＋シナリオパック」の 2 点のみ。共有 `qa-lab` がフローを所有できるなら新トップレベルコマンドは足さない。

## 用語と略称

- **QA** = Quality Assurance（品質保証・テスト）
- **E2E** = End-to-End（端から端までの結合テスト）
- **SUT** = System Under Test（テスト対象。ここでは被験の OpenClaw ボット）
- **driver / observer** = 入力を送る側 / 第三者トラフィックを観測する側
- **canary** = 最初に通す簡易疎通シナリオ
- **OTLP/OpenTelemetry** = 分散トレース/計装の標準

## 関連ページ

- [[concepts/qa-automation]] — 対応する概念ページ
- [[sources/concepts/qa-matrix]] — Matrix のライブトランスポートレーン
- [[components/gateway]] / [[channels/slack]]
