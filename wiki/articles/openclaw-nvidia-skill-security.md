---
type: article
source_kind: blog
source_url: https://openclaw.ai/blog/openclaw-nvidia-skill-security
source_path: raw/articles/OpenClaw Collaborates with NVIDIA for Stronger Agent Skill Security.md
lang: en
author: [Vincent Koc, Patrick Erichsen]
published: 2026-06-01
summarized: 2026-06-15
title: "OpenClaw、NVIDIA と協業しエージェントスキルセキュリティを強化"
tags: [clawhub, skills, security, skillspector, clawscan, nvidia, supply-chain]
related:
  - "[[components/clawhub]]"
  - "[[concepts/threat-model]]"
  - "[[concepts/skills]]"
translation: "[[articles/translations/openclaw-nvidia-skill-security]]"
---

# OpenClaw、NVIDIA と協業しエージェントスキルセキュリティを強化（要約）

> 原典: `raw/articles/OpenClaw Collaborates with NVIDIA for Stronger Agent Skill Security.md` ・ https://openclaw.ai/blog/openclaw-nvidia-skill-security
> 全文翻訳（一文一文の忠実訳）は [[articles/translations/openclaw-nvidia-skill-security]]
> ℹ️ OpenClaw 公式ブログの告知記事（二次資料）。著者: Vincent Koc / Patrick Erichsen、公開 2026-06-01。

## 一言まとめ

ClawHub（OpenClaw の Plugin/Skill マーケットプレイス）が、**公開前に全スキルを検証するゲート**を導入したという告知。NVIDIA と協業し、各スキルに来歴・能力・スキャン結果を記す **Skill Card** を付与し、AI 支援スキャナー **SkillSpector** で「マルウェアでない agentic リスク（隠し指示・過度な権限・宣言と挙動の不一致）」を検出、**ClawScan**（LLM-as-judge）が複数シグナルを総合して Clean/Suspicious/Malicious を判定する。

## 位置づけ（既存 wiki との接続）

これは [[components/clawhub]]（Plugin/Skill マーケットプレイス）の**セキュリティ強化の告知**で、[[concepts/threat-model]] が「第 5 の信頼境界＝サプライチェーン」として挙げた **T-PERSIST-001（悪意ある Skill の公開）** への直接の対策。[[concepts/skills]] が「サードパーティ Skill は信頼できないコードとして扱う」と述べる点を、レジストリ側の検証で補強する。既出の VirusTotal 連携（[[sources/security/threat-model-atlas]] が「進行中」としていた）に加え、本記事は SkillSpector / ClawScan / Skill Card という具体を明かす。

> ⚠️ **二次資料・新機能の告知**：Skill Card・SkillSpector・ClawScan・公開データセットはブログ時点の情報で、公式 docs（[[components/clawhub]] / [[concepts/skills]]）にはまだ反映されていない可能性がある。詳細は本ページに置き、concept/component 本文（公式 docs ベース）は書き換えない。

## 要点（記事の主張と新規情報）

- **インストール前に知りたい 3 点**：①何をすると主張するか ②同梱コードが主張と一致するか ③問題時の影響範囲（blast radius）。
- **ClawScan パイプライン**：新スキルバージョン公開時、OpenAI Codex エージェントが 3 つの独立スキャナーの出力（自前の静的解析・VirusTotal・NVIDIA SkillSpector）を受け取り、来歴/メタデータ/モデレーション履歴と総合して **Skill Card ＋判定（Clean/Suspicious/Malicious）** を生成。
- **NVIDIA Skill Cards**：オープンな信頼アーティファクト仕様。公開者の自己申告でなく **ClawHub が検証**。`openclaw skills verify <slug> --card` で参照。
- **NVIDIA SkillSpector**：静的＋AI セマンティック解析で agentic リスクを検出。発見は **advisory（自動ブロックしない）**、ClawScan が総合判断。
- **初期計測の発見**：3 スキャナーの陽性はほとんど重ならない（Jaccard 一致 ≤10.4%、3 つ全一致は 0.69%、陽性の 81.9% は単一スキャナー由来）。各スキャナーはリスク表面が異なる（VirusTotal＝マルウェア評判、静的解析＝危険コード、SkillSpector＝agentic リスク）。→ **LLM-as-judge（ClawScan）が必要**な理由。
- **データセット公開**：67,453 件のスキャン結果を Hugging Face（`OpenClaw/clawhub-security-signals`）でオープンソース化。コンパニオン論文あり。

## なぜ重要か

[[concepts/threat-model]] が最上位リスク（P0）に挙げた「悪意ある Skill の公開」は、従来のマルウェアスキャンでは捕らえきれない——「ログ要約と称してデータを外部送信」「誤フラグで本番を消す CLI に向ける」は**古典的マルウェアではない agentic リスク**だから。本記事の SkillSpector＋ClawScan は、この新しい攻撃面に「複数の異なるリスク表面を総合する LLM 判定」で対処する設計であり、[[components/clawhub]] が「サプライチェーン境界をどう守るか」の具体解になっている。

## 関連ページ

- [[articles/translations/openclaw-nvidia-skill-security]] — 全文翻訳
- [[components/clawhub]] / [[concepts/threat-model]] / [[concepts/skills]]
- [[sources/security/threat-model-atlas]]（T-PERSIST-001・VirusTotal） / [[concepts/security]]
