---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/multiple-gateways
source_path: raw/docs/gateway/multiple-gateways.md
doc_section: gateway
title: "複数の Gateway"
ingested: 2026-06-14
tags: [gateway, multiple-gateways, rescue-bot, profile, isolation, ports]
related:
  - "[[components/gateway]]"
  - "[[concepts/configuration]]"
  - "[[sources/gateway/gateway-lock]]"
---

# 複数の Gateway（解説）

> 原典: `raw/docs/gateway/multiple-gateways.md` ・ https://docs.openclaw.ai/ja-JP/gateway/multiple-gateways

## 一言まとめ

基本は **1 ホスト 1 Gateway**（単一 Gateway が複数接続・複数エージェントを捌ける）。より強い分離や冗長性（レスキューボット）が要るときだけ、**別プロファイル・別ポート**で複数の Gateway を走らせる方法。

## 位置づけ

[[components/gateway]] の「ホストごと 1 つ」が既定だが、それを意図的に破る運用パターン。一意性は [[sources/gateway/gateway-lock]] が強制し、各インスタンスの分離は [[concepts/configuration]] の状態ディレクトリ/ポート設定で行う。

## 仕組み・ふるまい

- **レスキューボット（推奨パターン）**：メインはデフォルトプロファイル、レスキューは `--profile rescue` で別 Telegram ボット・別ベースポート（例 `19789`）。メインが停止していてもデバッグ/設定変更ができる「DM ベースの復旧経路」。
- `openclaw --profile rescue onboard` は通常のオンボーディングを**別プロファイルに**書く：独自の設定ファイル・状態ディレクトリ・ワークスペース（既定 `~/.openclaw/workspace-rescue`）・管理サービス名。
- 一般的なマルチ Gateway も同じパターン（`openclaw --profile ops setup` ＋ `--profile ops gateway --port 19789`）。

## 設定・使い方の要点

- **分離チェックリスト（インスタンスごとに一意に）**：`OPENCLAW_CONFIG_PATH`・`OPENCLAW_STATE_DIR`・`agents.defaults.workspace`・`gateway.port`（`--port`）・派生するブラウザ/canvas/CDP ポート。
- **派生ポート**：ブラウザ制御 = ベース+2（loopback のみ）、canvas は Gateway HTTP と同ポート、CDP は `browser.controlPort + 9..+108` から自動割り当て。**ベースポート間は最低 20 空ける**。
- 確認：`openclaw gateway status --deep`（残存サービス検出）、`openclaw --profile rescue gateway probe`（`multiple reachable gateways` 警告は意図的な複数構成でのみ想定）。

## 注意点・落とし穴

- ⚠️ **`browser.cdpUrl` を複数インスタンスで同じ値に固定しない**。各インスタンスは独自のブラウザ制御ポートと CDP 範囲を持つ（明示するなら `browser.profiles.<name>.cdpPort`、リモート Chrome は `cdpUrl` をプロファイル/インスタンスごとに）。
- 状態ディレクトリ/設定/ワークスペース/ポートを共有すると設定競合・ポート衝突になる。

## 用語と略称

- **プロファイル（profile）** = 設定/状態/ワークスペース/サービス名を分ける名前付き構成（`--profile <name>`）
- **レスキューボット** = メイン障害時の復旧用に分離した別 Gateway
- **ベースポート / 派生ポート** = `gateway.port` と、そこから算出されるブラウザ/CDP 等のポート
- **CDP** = Chrome DevTools Protocol（ブラウザ制御）

## 関連ページ

- [[components/gateway]] / [[concepts/configuration]]
- [[sources/gateway/gateway-lock]] / [[sources/gateway/gateway]]
