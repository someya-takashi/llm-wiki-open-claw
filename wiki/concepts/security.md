---
type: concept
aliases: [Security, セキュリティ, threat model, 脅威モデル]
tags: [security, threat-model, hardening, prompt-injection, trust-boundary]
related:
  - "[[concepts/sandboxing]]"
  - "[[concepts/authentication]]"
  - "[[concepts/secrets]]"
  - "[[concepts/multi-agent]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/security]]"
  - "[[sources/gateway/operator-scopes]]"
  - "[[sources/gateway/security/audit-checks]]"
  - "[[sources/gateway/security/secure-file-operations]]"
updated: 2026-06-14
---

# セキュリティ

OpenClaw のセキュリティは **「Gateway ごとに 1 つの信頼済みオペレーター境界」＝単一ユーザーのパーソナルアシスタントモデル**を前提とする。**敵対的マルチテナント分離ではない**――混在/敵対ユーザーが要るなら信頼境界を分割する（別 Gateway＋認証情報、理想は別 OS ユーザー/ホスト）。

## なぜ重要か

AI エージェントに実マシン上のツール（シェル・ファイル・ブラウザ・メッセージング）を持たせる以上、**「何ができるか」を構造的に制限すること**が最大の安全策になる。中核思想は **「知能より前にアクセス制御」**――モデルの賢さや人格に頼らず、認証（[[concepts/authentication]]）・サンドボックス（[[concepts/sandboxing]]）・ツールポリシー・承認・許可リスト・シークレット外部化（[[concepts/secrets]]）で守る。特に**プロンプトインジェクション**（受信メッセージや外部コンテンツでモデルを誘導する攻撃）は、公開 DM が無くても起こり得るため、オープングループ＋exec/昇格のような高リスク経路を設計で塞ぐ。

## 押さえる点

- **脅威モデル**：1 Gateway = 1 信頼境界。`security.trust_model.multi_user_heuristic` 監査が「パーソナル前提なのにマルチユーザーに見える」を警告。
- **認可**：オペレータースコープ（`operator.read/write/admin/...`）が認証後にできることを制限（→ [[sources/gateway/operator-scopes]]）。
- **ファイル安全**：`@openclaw/fs-safe` のライブラリガードレール（→ [[sources/gateway/security/secure-file-operations]]）。実分離は OS 権限/サンドボックス。
- **検証**：`openclaw security audit` の `checkId` カタログ（→ [[sources/gateway/security/audit-checks]]）。重大例＝設定の world-readable・`bind_no_auth`・`hooks.token_reuse_gateway_token`・`open_groups_with_elevated`。
- **強化ベースライン**：ファイル権限・ネットワーク公開を絞る／SecretRef／redaction／読み取り専用ツール＋サンドボックス／メンション必須／マルチエージェントのアクセスプロファイル（[[concepts/delegate-architecture]]）／インシデント対応（封じ込め→ローテーション→監査）。

脅威モデルと 60 秒強化は [[sources/gateway/security]]。敵対的脅威の体系的な分析（MITRE ATLAS・形式検証）は [[concepts/threat-model]]、送信側のネットワーク制御（フォワードプロキシ・SSRF 強化）は [[sources/security/network-proxy]]。

## 関連

- [[concepts/sandboxing]] / [[concepts/exec]] / [[concepts/authentication]] / [[concepts/secrets]] / [[concepts/threat-model]]
- [[concepts/multi-agent]] / [[concepts/delegate-architecture]] / [[concepts/web-search]]（SSRF 面） / [[components/gateway]]
- 📝 ブログ（二次資料・外部）：メモリを「価値の高い攻撃面」とみなし蒸留/統合/注入の 3 段でガードレール（PII 拒否・コンテキストポイズニング/指示インジェクション対策・助言扱い）→ [[articles/context-engineering-personalization]]
