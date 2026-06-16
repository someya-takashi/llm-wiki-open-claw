---
type: component
aliases: [ClawHub, クローハブ, Plugin Marketplace, Skill Marketplace]
tags: [clawhub, marketplace, plugin, skill, distribution, moderation]
concepts:
  - "[[concepts/threat-model]]"
  - "[[concepts/security]]"
related:
  - "[[components/plugin-system]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/tools/plugin]]"
  - "[[sources/plugins/community]]"
  - "[[sources/plugins/manage-plugins]]"
updated: 2026-06-14
---

# ClawHub

**ClawHub** は OpenClaw の **Plugin / Skill マーケットプレイス**——外部 Plugin（と Skill）を公開・検索・配布する標準サーフェス。`openclaw plugins install clawhub:<package>` でインストールでき、裸のパッケージ指定も「まず ClawHub を確認、なければ npm」の順で解決される。[[components/plugin-system]] の外部エコシステムを支える配布層。

## 役割

- **配布と発見**：インストール前に検索可能なメタデータ・バージョン履歴・レジストリスキャン結果を確認できる（[[sources/plugins/manage-plugins]] の公開手順 `clawhub package publish`）。コミュニティ Plugin の掲載は ClawHub で行い、発見目的の docs PR は不要（[[sources/plugins/community]]）。
- **モデレーション/スキャン**：公開物にパターンベースのモデレーション・危険コードスキャン・GitHub アカウント年齢確認を適用（VirusTotal 連携は予定）。`clawhub package rescan <name>` で再チェックを依頼できる。

## セキュリティ上の位置づけ

ClawHub は [[concepts/threat-model]] の **5 番目の信頼境界＝サプライチェーン**にあたる。⚠️ Skill/Plugin は基本的にエージェント権限で動くため、悪意ある公開物（T-PERSIST-001）や更新汚染（T-PERSIST-002）が最上位級リスク。`--dangerously-force-unsafe-install` は自分のマシンのスキャン誤検知を超えるための非常口であり、ClawHub 側の再スキャンやブロック解除は行わない（[[concepts/security]]）。

## 位置づけ

> ℹ️ `clawhub/` セクションの専用ドキュメント（CLI・ダッシュボード等）は未取り込み。本ページは [[sources/tools/plugin]]・[[sources/plugins/community]]・[[sources/plugins/manage-plugins]] が言及する範囲から構成しており、専用 docs を ingest した際に拡充する。

## 関連

- [[concepts/threat-model]] / [[concepts/security]]
- [[components/plugin-system]] / [[components/gateway]]
- [[sources/tools/plugin]] / [[sources/plugins/community]] / [[sources/plugins/manage-plugins]]
- 📝 ブログ告知（二次資料）：NVIDIA 協業の公開前検証ゲート＝Skill Card / SkillSpector / ClawScan → [[articles/openclaw-nvidia-skill-security]]
