---
type: concept
aliases: [QA, QA automation, QA overview, E2E QA]
tags: [qa, testing, e2e, live-transport, maintainer]
related:
  - "[[sources/concepts/qa-matrix]]"
  - "[[concepts/mantis]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/qa-e2e-automation]]"
  - "[[sources/concepts/qa-matrix]]"
  - "[[sources/channels/qa-channel]]"
updated: 2026-06-14
---

# QA 自動化

OpenClaw の **QA（Quality Assurance）スタック**は、ユニットテストより現実的に・**実際のチャネルの形に近い方法**で製品を検証するためのメンテナー向け仕組み。合成チャネル（`qa-channel`、[[sources/channels/qa-channel]]）・デバッガー UI とバス（`qa-lab`）・実チャネルを駆動するライブトランスポートアダプター（[[sources/concepts/qa-matrix]] ほか）から成り、`pnpm openclaw qa <subcommand>` で動く。

## なぜ重要か

[[components/gateway]] とチャネル統合の「実トランスポート上での正しさ」は、ユニットテストでは届かない領域（mention gating・allowlist・スレッド・再起動 replay・E2EE・承認メタデータ配信など）にある。QA 自動化はそこを **ライブトランスポート契約**――Matrix/Telegram/Discord/Slack が共有する 1 つのチェックリスト――で回帰検証し、製品の信頼性を支える。`qa-channel` は別途、広範なプロダクト挙動の合成スイートとして維持される。

## 押さえる点

- **メンテナー専用 / ソースチェックアウト専用**：パッケージ済みリリースは QA Lab を省くため、利用者向け機能ではない。
- 代表コマンド：`qa suite`（リポジトリ管理シナリオ）・`qa coverage`・`qa character-eval`（複数ライブモデルで人格を判定）・`qa matrix|telegram|discord|slack`（ライブレーン）・`qa mantis`（→ [[concepts/mantis]]、修正前後のライブ検証）。
- 観測アーティファクトの本文は既定で伏せ字化。認証情報は環境変数か共有 **Convex プール**（排他リース）。

詳細（構成要素・コマンド・OTel スモーク・新チャネル追加基準）は [[sources/concepts/qa-e2e-automation]]、Matrix レーンは [[sources/concepts/qa-matrix]]。

## 関連

- [[sources/concepts/qa-matrix]] — Matrix のライブトランスポートレーン
- [[concepts/mantis]]（未作成） — 修正前後のライブ検証
- [[components/gateway]]
