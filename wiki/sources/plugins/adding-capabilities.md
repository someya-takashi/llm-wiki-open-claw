---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/adding-capabilities
source_path: raw/docs/plugins/adding-capabilities.md
doc_section: plugins
title: "機能の追加（コントリビューターガイド）"
ingested: 2026-06-14
tags: [plugin, capability, contributor, core, provider-seam, image-generation]
related:
  - "[[components/plugin-system]]"
  - "[[sources/plugins/building-plugins]]"
  - "[[concepts/agent-runtimes]]"
---

# 機能の追加（コントリビューターガイド）（解説）

> 原典: `raw/docs/plugins/adding-capabilities.md` ・ https://docs.openclaw.ai/ja-JP/plugins/adding-capabilities
>
> ℹ️ **OpenClaw コア開発者向け**のガイド。外部 Plugin を作るなら [[sources/plugins/building-plugins]]。

## 一言まとめ

画像生成・動画生成のような**新しい共有ドメイン（capability）を OpenClaw コアに足す**ときの手順。プロバイダー/ハーネスが差し込む「継ぎ目（seam）」をコアに用意する作業。

## 位置づけ

[[components/plugin-system]] の capability モデルをコア側から拡張する話で、[[concepts/agent-runtimes]] の provider/runtime の継ぎ目に関わる。個々の Plugin 実装（[[sources/plugins/building-plugins]]）の一段下の基盤。

## 仕組み・ふるまい

- **作るタイミング**：複数プロバイダーが実装しうる新ドメインが必要なとき（単発機能なら既存ツール/Plugin で足りる）。
- **標準的な順序**：capability 定義 → プロバイダー/ハーネスの継ぎ目 → ファイル配置 → 登録 → レビュー。実例として画像生成の追加手順。

## 設定・使い方の要点

- 何をどこに置くか・ファイルチェックリスト・レビューチェックリストに従う。詳細なアーキテクチャ（capability モデル/所有権/ロードパイプライン）は Plugin 内部構造リファレンス。

## 注意点・落とし穴

- 外部 Plugin 開発者には不要（コアの共有ドメイン追加専用）。プロバイダーとハーネスの継ぎ目を正しく切らないと、特定プロバイダーに密結合してしまう。

## 用語と略称

- **capability（ケイパビリティ）** = 複数プロバイダーが実装しうる共有機能ドメイン
- **seam（継ぎ目）** = プロバイダー/ハーネスが差し込む拡張点
- **コントリビューターガイド** = コア開発者向けの手引き

## 関連ページ

- [[components/plugin-system]] / [[sources/plugins/building-plugins]]
- [[concepts/agent-runtimes]] / [[sources/plugins/sdk-provider-plugins]]
