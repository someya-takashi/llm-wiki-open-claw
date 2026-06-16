---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/secrets-plan-contract
source_path: raw/docs/gateway/secrets-plan-contract.md
doc_section: gateway
title: "Secrets 適用プランコントラクト"
ingested: 2026-06-14
tags: [secrets, secrets-apply, contract, plan, validation]
related:
  - "[[concepts/secrets]]"
  - "[[sources/gateway/secrets]]"
---

# Secrets 適用プランコントラクト（解説）

> 原典: `raw/docs/gateway/secrets-plan-contract.md` ・ https://docs.openclaw.ai/ja-JP/gateway/secrets-plan-contract

## 一言まとめ

`openclaw secrets apply --from <plan.json>` が**強制する厳密な契約**――プランの各ターゲット（どの設定パスに SecretRef を書くか）が登録済みの形式に一致しなければ、**設定を変更する前に apply が失敗**する。

## 位置づけ

[[concepts/secrets]] の「適用（書き込み）」面の安全規約。[[sources/gateway/secrets]] の `secrets configure`/`apply` フローが生成・適用するプランの検証ルールを定義する。

## 仕組み・ふるまい

- **プランファイル**：`{ version, protocolVersion, targets: [...] }`。各ターゲットは `type`（例 `models.providers.apiKey`・`auth-profiles.api_key.key`）・`path`・任意の `pathSegments`・`providerId`/`accountId`/`agentId`・`ref`（SecretRef）。
- **パス検証ルール**：`type` は既知のターゲットタイプ／`path` は空でないドット区切り／`pathSegments` を指定するなら `path` と完全一致に正規化／**禁止セグメント `__proto__`/`prototype`/`constructor` を拒否**／正規化パスがターゲットタイプの登録形式に一致／`providerId`/`accountId` はパス内の id と一致／`auth-profiles.json` ターゲットは `agentId` 必須（新規マッピングは `authProfileProvider` も）。
- **失敗時**：`Invalid plan target path for ...` で終了し、**書き込みは一切コミットしない**。
- サポートされるターゲットスコープは [SecretRef Credential Surface] に一覧（互換エイリアス `models.providers.apiKey`/`skills.entries.apiKey`/`channels.googlechat.serviceAccount` も受理）。

## 設定・使い方の要点

- 検証→適用：`secrets apply --from <plan> --dry-run` → `secrets apply --from <plan>`。exec を含むプランは **`--allow-exec` を dry-run と書き込みの両方で**明示（無いと書き込みモードで拒否）。
- 無効ターゲットで失敗したら `openclaw secrets configure` で再生成するか、ターゲットパスをサポート形式に直す。
- ref のみの `auth-profiles.json` エントリ（`keyRef`/`tokenRef`）はランタイム解決・監査対象に含まれる。

## 注意点・落とし穴

- これは**書き込み前のゲート**。無効なら 1 ターゲットでも全体が止まり、ディスク設定もランタイムも変わらない（[[sources/gateway/secrets]] の一方向安全ポリシーと整合）。
- プロトタイプ汚染対策として禁止セグメントを必ず拒否する。

## 用語と略称

- **プラン（plan）** = 適用する SecretRef ターゲットの集合を記述した JSON
- **ターゲット（target）** = SecretRef を書き込む設定上の場所（type＋path）
- **dry-run** = 書き込まずに検証だけ行う実行
- **プロトタイプ汚染** = `__proto__` 等を悪用したオブジェクト改竄攻撃

## 関連ページ

- [[concepts/secrets]] — 対応する概念ページ
- [[sources/gateway/secrets]] — SecretRef 管理の全体
