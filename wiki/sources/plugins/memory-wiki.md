---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/memory-wiki
source_path: raw/docs/plugins/memory-wiki.md
doc_section: plugins
title: "Memory Wiki"
ingested: 2026-06-14
tags: [plugin, memory, wiki, knowledge-vault, claims, obsidian, compile]
related:
  - "[[concepts/memory]]"
  - "[[concepts/active-memory]]"
  - "[[components/plugin-system]]"
---

# Memory Wiki（解説）

> 原典: `raw/docs/plugins/memory-wiki.md` ・ https://docs.openclaw.ai/ja-JP/plugins/memory-wiki

## 一言まとめ

永続メモリを**ナビゲート可能な「知識 vault（wiki）」にコンパイル**する同梱 Plugin。決定論的なページ・構造化された主張（claim）と出典・ダッシュボード・機械可読ダイジェストを生成する。OpenClaw 自身を支える「LLM Wiki」パターンをメモリに適用したもの。

## 位置づけ

[[concepts/memory]] の上に重なる知識レイヤー。**Active Memory（[[concepts/active-memory]]）を置き換えない**——リコール/昇格/インデックス/Dreaming は Active Memory が担い続け、`memory-wiki` はその横で永続知識を wiki 化する。Plugin 機構は [[components/plugin-system]]。

## 仕組み・ふるまい

- **Vault モード**：`isolated`（独立 vault）／`bridge`（既存メモリと橋渡し）／`unsafe-local`。決定論的な vault レイアウト・構造化された主張＋証拠メタデータ・エージェント向けエンティティメタデータを持つ。
- **コンパイルパイプライン**：散在するメモリ → ページ・主張・ダッシュボード・健全性レポートにコンパイル。検索/取得とプロンプト/コンテキストへの注入を制御。

## 設定・使い方の要点

- 推奨ハイブリッド：QMD ＋ bridge モード（[[sources/concepts/memory-qmd]]）。`plugins.entries.memory-wiki` 設定、CLI、**Obsidian サポート**（vault を Obsidian で開ける）。
- アクティブメモリスロットは別 Plugin（例 memory-lancedb）が持ち、memory-wiki はコンパニオンとして併用。

## 注意点・落とし穴

- 「Markdown の山」でなく「保守された知識レイヤー」が欲しい時に使う。Active Memory の役割（想起/昇格）とは責務が分かれている点に注意。

## 用語と略称

- **vault** = 知識を束ねたファイル群（Obsidian の用語）
- **主張（claim）** = 出典付きで構造化された知識の単位
- **コンパイル（compile）** = 生メモリを構造化ページ群に変換すること
- **QMD** = ローカル優先のサイドカーメモリ（[[sources/concepts/memory-qmd]]）
- **Dreaming** = 短期→長期メモリの昇格（[[concepts/dreaming]]）

## 関連ページ

- [[concepts/memory]] / [[concepts/active-memory]] / [[concepts/dreaming]]
- [[sources/plugins/memory-lancedb]] / [[components/plugin-system]]
