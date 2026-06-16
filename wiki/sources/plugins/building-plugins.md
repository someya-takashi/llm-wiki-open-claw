---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/building-plugins
source_path: raw/docs/plugins/building-plugins.md
doc_section: plugins
title: "Plugin の構築"
ingested: 2026-06-14
tags: [plugin, sdk, building, register, tool, cli, manifest]
related:
  - "[[components/plugin-system]]"
  - "[[components/clawhub]]"
  - "[[sources/tools/plugin]]"
---

# Plugin の構築（解説）

> 原典: `raw/docs/plugins/building-plugins.md` ・ https://docs.openclaw.ai/ja-JP/plugins/building-plugins

## 一言まとめ

ネイティブ Plugin を作る入門ガイド。チャネル・モデルプロバイダー・音声・メディア・ツールなどの機能を `register(api)` で登録するパッケージの作り方と、提出前チェックリストを示す。

## 位置づけ

[[components/plugin-system]] の作成者向け入口（インストール/管理は [[sources/tools/plugin]]）。種類別の詳細はチャネル（[[sources/plugins/sdk-channel-plugins]]）・プロバイダー（[[sources/plugins/sdk-provider-plugins]]）・CLI バックエンド（[[sources/plugins/cli-backend-plugins]]）へ。公開は [[components/clawhub]]。

## 仕組み・ふるまい

- **どの種類か**で分岐：チャネル Plugin / プロバイダー Plugin / ツール Plugin / 音声・メディア等。OpenClaw リポジトリに入れる必要はなく、ClawHub/npm から配布。
- **ツール Plugin クイックスタート**：`definePluginEntry({ id, name, register(api) { api.registerTool(...) } })`。`api.registerTool` でスキーマ＋ハンドラを登録、`api.registerCli`/`registerCommand` で CLI コマンドを足す。
- **インポート規約**：エントリのトップレベルは軽量・副作用なしに保つ（ネットワーク/サブプロセス/認証情報読み取りは `registrationMode === "full"` の背後へ）。

## 設定・使い方の要点

- パッケージ：`package.json` に `openclaw.extensions`（ソース）＋`runtimeExtensions`（ビルド済み JS）。マニフェスト（`openclaw.plugin.json` / [[sources/plugins/sdk-channel-plugins]] 参照）。
- 提出前チェックリスト：ランタイム登録の検証（`openclaw plugins inspect <id> --runtime`）、依存同梱、ドキュメント。

## 注意点・落とし穴

- フックで会話ライフサイクルに介入するなら [[sources/plugins/hooks]]。コア開発者として新ドメイン（capability）を足すなら [[sources/plugins/adding-capabilities]]。

## 用語と略称

- **SDK** = Software Development Kit（Plugin 開発キット）
- **`register(api)`** = Plugin が機能を登録するエントリ関数
- **マニフェスト（manifest）** = `openclaw.plugin.json`（Plugin のメタデータ）
- **registrationMode** = full/discovery 等、エントリが読み込まれる理由

## 関連ページ

- [[components/plugin-system]] / [[components/clawhub]] / [[sources/tools/plugin]]
- [[sources/plugins/sdk-channel-plugins]] / [[sources/plugins/sdk-provider-plugins]] / [[sources/plugins/cli-backend-plugins]] / [[sources/plugins/hooks]]
