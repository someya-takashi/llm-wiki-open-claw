---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/security
source_path: raw/docs/gateway/security.md
doc_section: gateway
title: "セキュリティ"
ingested: 2026-06-14
tags: [security, threat-model, hardening, prompt-injection, trust-boundary, audit]
related:
  - "[[concepts/security]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/authentication]]"
---

# セキュリティ（解説）

> 原典: `raw/docs/gateway/security.md` ・ https://docs.openclaw.ai/ja-JP/gateway/security
>
> ℹ️ OpenClaw の**包括的セキュリティガイド**（大）。本解説は全項目を転記せず、脅威モデルと強化の要点を地図として示す。

## 一言まとめ

OpenClaw のセキュリティ前提・脅威モデル・強化ベースラインを定義するハブ文書。前提は **「Gateway ごとに 1 つの信頼済みオペレーター境界」＝単一ユーザーのパーソナルアシスタントモデル**で、敵対的マルチテナント境界ではない。

## 位置づけ

[[concepts/security]] の本体。混同しやすい認証（[[concepts/authentication]]）・サンドボックス（[[concepts/sandboxing]]）・シークレット（[[concepts/secrets]]）・スコープ（[[sources/gateway/operator-scopes]]）を 1 つの脅威モデルに束ね、`openclaw security audit`（→ [[sources/gateway/security/audit-checks]]）で検証する。

## 仕組み・ふるまい（脅威モデルの要点）

- **信頼モデル**：1 Gateway = 1 信頼境界。混在/敵対ユーザーが要るなら**信頼境界を分割**（別 Gateway＋認証情報、理想は別 OS ユーザー/ホスト）。
- **中核思想「知能より前にアクセス制御」**：モデルの賢さに頼らず、ツールポリシー・サンドボックス・承認・許可リストで**できることを構造的に制限**する。
- **コンテキスト可視性 / コマンド認可モデル**：誰の入力が・どのエージェントに届き・何を起動できるかを設計で固定。
- **プロンプトインジェクション**：受信メッセージ・外部コンテンツは**信頼できない入力**。特殊トークンのサニタイズ、`allowUnsafeExternalContent` 等のバイパスは既定オフ。**公開 DM はインジェクションに不要**（外部コンテンツ経由でも起こる）。オープングループ＋exec/昇格は高リスク経路。
- **DM アクセスモデル**：`dmPolicy`（pairing/allowlist/open/disabled）＋マルチユーザー DM 分離（セキュア DM モード推奨、`session.dmScope`）。グループは全所で**メンション必須**。

## 設定・使い方の要点（60 秒強化ベースライン）

- **ファイル権限**：`~/.openclaw`・`openclaw.json`・`auth-profiles.json` を他者が読み書きできないように（→ audit が検出）。
- **ネットワーク公開**：bind/port/firewall を絞る。`gateway.auth` 必須、Tailscale Funnel（公開）は重大。Control UI 非ループバックは `allowedOrigins` 明示。
- **シークレット**：config 直書きでなく SecretRef（[[concepts/secrets]]）。ログ/トランスクリプトは redaction＋保持に注意。
- **ツール/サンドボックス**：読み取り専用モード・最小ツールサーフェス・サンドボックス化（[[concepts/sandboxing]]）。サブエージェント委任・ブラウザ SSRF ポリシー（既定厳格）にガードレール。
- **マルチエージェントのアクセスプロファイル**：フルアクセス/読み取り専用/FS・シェルなし等を `agents.list[].tools`/`sandbox` で（→ [[concepts/delegate-architecture]]）。
- **インシデント対応**：封じ込め → ローテーション（漏えい時は侵害を想定）→ 監査 → レポート収集。シークレットスキャンも。

## 注意点・落とし穴

- ⚠️ **OpenClaw は敵対的マルチテナント分離ではない**。`security.trust_model.multi_user_heuristic` 監査が「パーソナル前提なのに設定がマルチユーザーに見える」を警告する。
- `hooks.token` を Gateway トークンと共用しない（重大）。Node の影響大コマンド（カメラ/画面/SMS 等）は許可制。
- 詳細な checkId は [[sources/gateway/security/audit-checks]]、安全なファイル操作は [[sources/gateway/security/secure-file-operations]]。

## 用語と略称

- **脅威モデル** = 何から守るか・守らないかの前提
- **信頼境界** = 同じ信頼レベルとみなす範囲
- **プロンプトインジェクション** = 受信コンテンツでモデルを誘導する攻撃
- **SSRF** = Server-Side Request Forgery（サーバーに不正リクエストを行わせる攻撃）
- **マルチテナント** = 複数の独立利用者が 1 システムを共有する形態

## 関連ページ

- [[concepts/security]] — 対応する概念ページ
- [[sources/gateway/security/audit-checks]] / [[sources/gateway/security/secure-file-operations]]
- [[concepts/sandboxing]] / [[concepts/authentication]] / [[concepts/secrets]] / [[sources/gateway/operator-scopes]]
