---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/auth-credential-semantics
source_path: raw/docs/auth-credential-semantics.md
doc_section: root
title: "認証情報のセマンティクス"
ingested: 2026-06-14
tags: [authentication, credentials, probe, reason-codes, oauth, portability]
related:
  - "[[concepts/authentication]]"
  - "[[sources/gateway/authentication]]"
  - "[[concepts/oauth]]"
---

# 認証情報のセマンティクス（解説）

> 原典: `raw/docs/auth-credential-semantics.md` ・ https://docs.openclaw.ai/ja-JP/auth-credential-semantics
>
> ℹ️ これは docs の**トップレベル（ルート）ドキュメント**（URL に section プレフィックスが無い）。`raw/docs/` 直下に置く。

## 一言まとめ

OpenClaw 全体で共通の**認証情報の適格性（eligibility）と解決セマンティクス**を定義する正規ドキュメント。`models status --probe` の安定した**理由コード**、トークンの有効性ルール、エージェント間のプロファイル移植性、OAuth の SecretRef ガードを規定し、「選択時の挙動」と「ランタイムの挙動」を一致させる。

## 位置づけ

[[concepts/authentication]] のモデル認証の「正確な判定基準」面。[[sources/gateway/authentication]] が UI/操作を、本ページが**裏側のルール**（なぜそのプロファイルが使われた/除外されたか）を担う。OAuth の扱いは [[concepts/oauth]] とも接続する。

## 仕組み・ふるまい

### 安定したプローブ理由コード

`ok` / `excluded_by_auth_order` / `missing_credential` / `invalid_expires` / `expired` / `unresolved_ref` / `no_model`。

### トークン認証情報（`type: "token"`）

- `token` と `tokenRef` の一方/両方を持てる。両方無ければ不適格。
- `expires` は任意。あれば **0 より大きい有限数**でなければならず、無効値は `invalid_expires`、過去なら `expired`。`tokenRef` は `expires` 検証をバイパスしない。
- 解決ルールは `expires` 適格性と一致。解決できない参照は `unresolved_ref`。

### エージェントコピーの移植性

エージェント認証継承は**読み取り透過**（ローカルプロファイルが無ければデフォルト/メインから解決、複製しない）。`openclaw agents add` などの明示コピーでは：`api_key`/`token` は `copyToAgents: false` でない限り移植可、**`oauth` は既定で移植不可**（更新トークンが単回使用/ローテーション敏感のため。安全と分かっている場合のみ `copyToAgents: true`）。

### 設定専用の認証ルートと OAuth ガード

`mode: "aws-sdk"` の `auth.profiles` エントリは**ルーティングメタデータ**（保存認証情報ではない）。`auth-profiles.json` に `type: "aws-sdk"` を書かない（doctor が `auth.profiles` へ移動）。**OAuth SecretRef ポリシー**：SecretRef は静的認証情報専用で、`type/mode: "oauth"` のプロファイルでは `keyRef`/`tokenRef` を拒否（起動/リロードでハードエラー）。

## 設定・使い方の要点

- **明示的な認証順序フィルタ**：`auth.order.<provider>` を設定すると、`models status --probe` はその順序に残るプロファイルだけをプローブし、省かれたものは `excluded_by_auth_order`（暗黙に試行しない）。
- **外部 CLI 認証情報の検出**：呼び出し元は検出モードを選ぶ（`none`/`existing`/`scoped`）。読み取り専用パスは `allowKeychainPrompt: false`（macOS Keychain を読まない）。

## 注意点・落とし穴

- **レガシー互換**：プローブエラーの 1 行目 `Auth profile credentials are missing or expired.` は変えない（スクリプト互換）。人間向け詳細と安定理由コードは後続行に足す。
- 理由コードはスクリプトで判定に使える安定 API（メッセージ文言ではなくコードで分岐する）。

## 用語と略称

- **適格性（eligibility）** = その認証情報がプローブ/使用の対象になるか
- **理由コード（reason code）** = プローブ結果の安定した機械可読ラベル
- **移植性（portability）** = エージェント間で認証情報をコピーしてよいか
- **読み取り透過継承** = 複製せずに親ストアの認証情報を参照する仕組み

## 関連ページ

- [[concepts/authentication]] — 認証の全体像
- [[sources/gateway/authentication]] / [[concepts/oauth]] / [[concepts/secrets]]
