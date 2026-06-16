---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/security/formal-verification
source_path: raw/docs/security/formal-verification.md
doc_section: security
title: "形式検証（セキュリティモデル）"
ingested: 2026-06-14
tags: [formal-verification, tla-plus, model-checking, security, regression]
related:
  - "[[concepts/threat-model]]"
  - "[[concepts/security]]"
  - "[[concepts/pairing]]"
---

# 形式検証（セキュリティモデル）（解説）

> 原典: `raw/docs/security/formal-verification.md` ・ https://docs.openclaw.ai/ja-JP/security/formal-verification

## 一言まとめ

OpenClaw の意図したセキュリティポリシー（認可・セッション分離・ツールゲーティング・誤設定安全性）を **TLA+/TLC で機械検証**する、攻撃者主導の「セキュリティ回帰スイート」。各主張に有限状態のモデルチェックと、バグを再現する**否定モデル**がある。

## 位置づけ

[[concepts/threat-model]] を「実行可能な検証」で裏打ちする面。検証対象は [[concepts/pairing]]・[[concepts/security]]・ルーティング分離（[[concepts/session]]）など本 wiki の中核不変条件。

## 仕組み・ふるまい（検証されている主張）

- **Gateway 公開/誤設定**：認証なしで loopback を超えてバインドするとリモート侵害リスク、token/password は未認証攻撃者をブロック（`make gateway-exposure-v2[-protected|-negative]`）。
- **Node 実行パイプライン**（最高リスク）：`exec host=node` は許可リスト＋宣言コマンド＋ライブ承認が必要、承認はトークン化でリプレイ防止（`make nodes-pipeline`/`approvals-token`）。
- **ペアリングストア**：TTL（Time To Live, 有効期限）と保留上限を尊重、並行下でも `MaxPending`・冪等性を強制（`make pairing`/`pairing-race`/`pairing-idempotency`）。
- **受信ゲーティング**：メンション必須グループで未認可の制御コマンドはゲートをバイパスできない（`make ingress-gating`）。
- **ルーティング/セッション分離**：明示リンクなしに別ピアの DM は同セッションに混ざらない、dmScope 優先順位＋identityLinks（`make routing-isolation`/`routing-precedence`）。受信トレース相関とリトライ冪等性も検証。

## 設定・使い方の要点

- モデルは別リポジトリ `vignesh07/openclaw-formal-models`（クローンして Java 11+ の TLC を `make <target>` で実行）。「グリーン」＝主張成立、「レッド」＝否定モデルが想定どおり反例を出す。

## 注意点・落とし穴

- ⚠️ これは**モデルであり TypeScript 実装全体の証明ではない**。モデルとコードのずれ、TLC が探索した状態空間の範囲、明示的な環境前提（正しいデプロイ/設定）に結果は依存する。「全面的にセキュア」を意味しない。

## 用語と略称

- **TLA+ / TLC** = 並行システムの仕様記述言語と、その有限状態モデルチェッカー
- **モデルチェック（model checking）** = 状態空間を網羅探索して性質の成立を確認
- **否定モデル（negative model）** = バグを意図的に再現して反例を出すモデル
- **TTL** = Time To Live（有効期限）
- **冪等性（idempotency）** = 同じ操作を繰り返しても結果が変わらない性質

## 関連ページ

- [[concepts/threat-model]] / [[sources/security/threat-model-atlas]]
- [[concepts/security]] / [[concepts/pairing]] / [[concepts/session]]
