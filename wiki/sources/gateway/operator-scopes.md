---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/operator-scopes
source_path: raw/docs/gateway/operator-scopes.md
doc_section: gateway
title: "オペレーターのスコープ"
ingested: 2026-06-14
tags: [operator-scopes, rbac, roles, control-plane, pairing, security]
related:
  - "[[concepts/security]]"
  - "[[concepts/authentication]]"
  - "[[concepts/pairing]]"
---

# オペレーターのスコープ（解説）

> 原典: `raw/docs/gateway/operator-scopes.md` ・ https://docs.openclaw.ai/ja-JP/gateway/operator-scopes

## 一言まとめ

オペレータースコープは、認証後の Gateway クライアントが**何を実行できるか**を定義する**コントロールプレーンのガードレール**。1 つの信頼済みオペレーター境界内の権限管理であり、敵対的マルチテナント分離ではない（強い分離は別 Gateway/OS ユーザー）。

## 位置づけ

[[concepts/authentication]]（接続認証）の後段にあたる**認可（何ができるか）**の層で、[[concepts/security]] の信頼モデルの一部。[[concepts/pairing]] のデバイス承認がスコープの永続的な源になる。

## 仕組み・ふるまい

- **ロール**（接続時に 1 つ）：`operator`（CLI/Control UI/自動化）/`node`（`node.invoke` でコマンドを公開する端末）。オペレーター RPC は `operator` ロール、Node 起点メソッドは `node` ロールが必要。
- **スコープレベル**：`operator.read`（読み取り）/`operator.write`（変更系。read も満たす）/`operator.admin`（管理：設定変更・更新・ネイティブフック・高リスク承認。全 `operator.*` を満たす）/`operator.pairing`（デバイス/Node ペアリング管理）/`operator.approvals`（exec/Plugin 承認）/`operator.talk.secrets`。未知の将来スコープは `operator.admin` 以外は完全一致が必要。
- **メソッドスコープは最初のゲートにすぎない**：各 RPC には最小権限のメソッドスコープがあり、ハンドラ到達後に対象に応じてより厳格なチェックが入る（例：`chat.send` は write だが永続 `/config set/unset` はコマンドレベルで `operator.admin`）。
- **デバイスペアリング承認**：ペアリングレコードが承認済みロール/スコープの永続的な源。**ペアリング済みデバイスが密かに広いアクセスを得ることはない**（広い要求は新たな保留アップグレードを作る）。承認には「呼び出し元がそのスコープか `operator.admin` を持つ」ことが要る。非管理者は自分のエントリのみ管理（自己スコープ）。

## 設定・使い方の要点

- **共有シークレット認証は信頼済みオペレーター扱い**：OpenAI 互換 HTTP と `/tools/invoke` は、より狭いスコープ宣言を送っても通常の完全なオペレーター既定スコープに復元される。ID 付きモード（trusted-proxy・private-ingress `none`）では宣言スコープを尊重できる。
- **Node ペアリング**で導出される追加スコープ：コマンドなし＝`operator.pairing`／非 exec Node コマンド＝`+operator.write`／`system.run`・`system.which`＝`+operator.admin`。Node ペアリングは ID/信頼を確立するが、Node 独自の `system.run` exec 承認ポリシーは置き換えない。

## 注意点・落とし穴

- これは**信頼境界の分離ではない**。実際の分離には別 OS ユーザー/ホストの別 Gateway を使う。
- trusted-proxy の `x-openclaw-scopes` でスコープを宣言できる（→ [[sources/gateway/trusted-proxy-auth]]）。

## 用語と略称

- **オペレータースコープ** = 認証後にできる操作の範囲（RBAC 的なガードレール）
- **コントロールプレーン** = Gateway を操作・設定する制御面
- **ロール** = 接続種別（operator / node）
- **RBAC** = Role-Based Access Control（ロールベースアクセス制御）

## 関連ページ

- [[concepts/security]] / [[concepts/authentication]] / [[concepts/pairing]]
- [[sources/gateway/trusted-proxy-auth]] / [[sources/gateway/security]]
