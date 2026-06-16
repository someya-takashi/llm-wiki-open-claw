---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/community
source_path: raw/docs/plugins/community.md
doc_section: plugins
title: "コミュニティ Plugin"
ingested: 2026-06-14
tags: [plugin, community, clawhub, third-party, catalog]
related:
  - "[[components/plugin-system]]"
  - "[[components/clawhub]]"
  - "[[sources/plugins/manage-plugins]]"
---

# コミュニティ Plugin（解説）

> 原典: `raw/docs/plugins/community.md` ・ https://docs.openclaw.ai/ja-JP/plugins/community

## 一言まとめ

新しいチャネル・ツール・プロバイダー等で OpenClaw を拡張する**サードパーティ製 Plugin** のカタログと提出ガイド。コミュニティが構築・保守し、通常は [[components/clawhub]] で公開され、1 コマンドでインストールできる。

## 位置づけ

[[components/plugin-system]] の外部エコシステム面。配布の標準発見サーフェスは [[components/clawhub]]、インストール手順は [[sources/plugins/manage-plugins]]。

## 仕組み・ふるまい

- `openclaw plugins install clawhub:<package>`（npm ホストは `install <package>`）。発見目的だけの docs PR は不要——ClawHub で公開する。
- **掲載例**：Apify（2 万以上のスクレイパー）・Codex App Server Bridge・DingTalk（企業ロボット）・Lossless Claw（DAG ベースの Compaction）・Opik（トレースエクスポート）・Prometheus Avatar（Live2D アバター）・QQbot・WeCom・Yuanbao（いずれも中国系チャネル）。

## 設定・使い方の要点

- **提出基準**：ClawHub/npm で公開（`openclaw plugins install <name>` が動く）・公開 GitHub リポジトリ・セットアップ/使用ドキュメント・継続保守。労力の少ないラッパーや保守放棄は却下され得る。

## 注意点・落とし穴

- 一部（QQbot 等）は現行リリースに**バンドル済み**で、通常は同梱版を使う（外部 Plugin は意図的にスタンドアロンを使いたい場合のみ）。

## 用語と略称

- **コミュニティ Plugin** = サードパーティ製の外部拡張
- **ClawHub** = OpenClaw の Plugin マーケットプレイス
- **DAG** = Directed Acyclic Graph（有向非巡回グラフ。会話要約構造）

## 関連ページ

- [[components/plugin-system]] / [[components/clawhub]]
- [[sources/plugins/manage-plugins]] / [[sources/tools/plugin]]
