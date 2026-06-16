---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/manage-plugins
source_path: raw/docs/plugins/manage-plugins.md
doc_section: plugins
title: "Plugin を管理する"
ingested: 2026-06-14
tags: [plugin, install, update, uninstall, publish, clawhub, npm]
related:
  - "[[components/plugin-system]]"
  - "[[components/clawhub]]"
  - "[[sources/tools/plugin]]"
---

# Plugin を管理する（解説）

> 原典: `raw/docs/plugins/manage-plugins.md` ・ https://docs.openclaw.ai/ja-JP/plugins/manage-plugins

## 一言まとめ

Plugin のインストール・一覧・更新・アンインストール・公開のコピペ可能な手順集。ほとんどのワークフローは「検索→インストール→Gateway 再起動→検証→不要ならアンインストール」で完結する。

## 位置づけ

[[components/plugin-system]] の運用手順（概要は [[sources/tools/plugin]]）。配布面は [[components/clawhub]]（公開ディスカバリ）と npm/git。

## 仕組み・ふるまい

- **一覧**：`openclaw plugins list [--enabled|--verbose|--json]`（`--json` に `dependencyStatus`）。コールドインベントリで、実行中 Gateway がランタイムをインポート済みかは別。
- **インストール**：`openclaw plugins search "<q>"` → `install <package>`（裸指定は ClawHub→npm）／`clawhub:`・`npm:`・`git:`・ローカルパス／`--link`（コピーせず開発用）。バージョン/タグ固定 `@1.2.3`/`@beta`。インストール後 `openclaw gateway restart` ＋ `inspect <id> --runtime --json` で登録を確認。
- **更新/削除**：`update <id|spec>`/`--all`、`uninstall <id> [--dry-run|--keep-files]`（設定・索引・allow/deny を削除）。

## 設定・使い方の要点

- **公開**：ClawHub（`clawhub package publish <org>/<plugin>`、メタデータ/バージョン/スキャン結果が見える主経路）または npm（`package.json` に `openclaw.extensions`＋`npm publish`）。同名が両方にあれば `npm:` は npm 解決を強制。
- ソース選択：ClawHub（ネイティブ発見/スキャン）／npm（既存 JS パッケージ・dist-tag）／Git（ブランチ/タグ/コミット）／ローカル（開発）。

## 注意点・落とし穴

- ⚠️ **Nix モード（`OPENCLAW_NIX_MODE=1`）では Plugin ライフサイクル操作が無効**（Nix ソースで管理）。`@beta` 等のタグは更新時に再利用、明示 spec を渡すと追跡先が切替。

## 用語と略称

- **dist-tag** = npm のバージョン別名（`latest`/`beta` 等）
- **ClawHub** = OpenClaw の Plugin マーケットプレイス
- **`--link`** = コピーせずソースパスを使う開発インストール
- **dependencyStatus** = Plugin の依存解決状況

## 関連ページ

- [[components/plugin-system]] / [[components/clawhub]]
- [[sources/tools/plugin]] / [[sources/plugins/community]] / [[sources/plugins/bundles]]
