---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/security/CONTRIBUTING-THREAT-MODEL
source_path: raw/docs/security/contributing-threat-model.md
doc_section: security
title: "脅威モデルへの貢献"
ingested: 2026-06-14
tags: [threat-model, contributing, mitre-atlas, process, community]
related:
  - "[[concepts/threat-model]]"
  - "[[sources/security/threat-model-atlas]]"
---

# 脅威モデルへの貢献（解説）

> 原典: `raw/docs/security/contributing-threat-model.md` ・ https://docs.openclaw.ai/ja-JP/security/CONTRIBUTING-THREAT-MODEL

## 一言まとめ

脅威モデルに**脅威・緩和策・攻撃チェーンを追加する貢献ガイド**。セキュリティ専門家でなくてよく、シナリオを自分の言葉で説明すれば、ATLAS マッピング・脅威 ID・リスク評価はメンテナーが付与する。

## 位置づけ

[[concepts/threat-model]] の運用面（[[sources/security/threat-model-atlas]] の「生きたドキュメント」を回す手順）。脆弱性報告とは別物。

## 仕組み・ふるまい

- **貢献方法**：脅威追加（`openclaw/trust` で issue）／緩和策提案（具体的・実行可能に。例「送信者ごと 10 msg/min」）／攻撃チェーン提案／既存修正（PR、issue 不要）。
- **脅威 ID カテゴリ**：RECON/ACCESS/EXEC/PERSIST/EVADE/DISC/EXFIL/IMPACT。リスクレベル：重大/高/中/低。ID とレベルはレビュー時に付与。
- **レビュー**：トリアージ（48h）→評価（ATLAS マッピング＋ID＋リスク）→ドキュメント化→マージ。

## 注意点・落とし穴

- ⚠️ **これは脅威モデルへの追加であって、実在脆弱性の報告ではない**。悪用可能な脆弱性は責任ある開示（Trust ページ）へ。

## 用語と略称

- **MITRE ATLAS** = AI システム向けの敵対的脅威フレームワーク
- **脅威 ID** = `T-EXEC-003` のような分類付き識別子
- **攻撃チェーン（attack chain）** = 複数脅威を繋いだ現実的攻撃シナリオ
- **責任ある開示（responsible disclosure）** = 脆弱性を安全に報告する手続き

## 関連ページ

- [[concepts/threat-model]] / [[sources/security/threat-model-atlas]] / [[sources/security/formal-verification]]
