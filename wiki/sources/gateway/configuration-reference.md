---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/configuration-reference
source_path: raw/docs/gateway/configuration-reference.md
doc_section: gateway
title: "設定リファレンス"
ingested: 2026-06-14
tags: [configuration, reference, schema, openclaw.json]
related:
  - "[[concepts/configuration]]"
  - "[[sources/gateway/configuration]]"
---

# 設定リファレンス（解説）

> 原典: `raw/docs/gateway/configuration-reference.md` ・ https://docs.openclaw.ai/ja-JP/gateway/configuration-reference
>
> ℹ️ これは `openclaw.json` の**網羅的フィールドリファレンス**（巨大）。本解説は全フィールドを転記せず、**トップレベルの地図**として要点を示す。正確な定義は `openclaw config schema` / `config.schema.lookup` が一次情報源。

## 一言まとめ

`~/.openclaw/openclaw.json` の主要設定サーフェスをフィールド単位で網羅するリファレンス。チャネルや Plugin が所有するコマンドカタログ、深いメモリ/QMD 調整は各専用ページに委ねる。

## 位置づけ

[[concepts/configuration]] の「全フィールド辞書」。タスク指向のガイドは [[sources/gateway/configuration]]、ドメイン別の詳細は config-agents/channels/tools の各ページ。**コード上の正は `openclaw config schema`（ライブ JSON Schema）と `config.schema.lookup`**で、ドキュメントのベースラインは `pnpm config:docs:check` で検証される。

## 仕組み・ふるまい（トップレベル・セクション地図）

- **channels** — チャネルごとの設定（→ [[sources/gateway/config-channels]]）。
- **agents / agents.defaults / agents.list / multiAgent / session / messages / talk** — エージェント・セッション・メッセージ（→ [[sources/gateway/config-agents]]）。
- **tools / models.providers** — ツールとカスタムプロバイダー（→ [[sources/gateway/config-tools]]）。
- **models** — モデルカタログ・フォールバック。**mcp** — MCP（Model Context Protocol）サーバー。**skills** — Skills 設定。**plugins** — Plugin（slots/entries、Codex ハーネス設定含む）。
- **commitments** — 推論コミットメント（→ [[concepts/commitments]]）。**browser** — ブラウザツール。**ui** — Control UI。
- **gateway** — サーバー（port/bind/auth/TLS/reload/OpenAI 互換/複数インスタンス分離）。**hooks** — Webhook（Gmail 連携含む）。**canvas** — Canvas Plugin ホスト。
- **discovery** — mDNS(Bonjour)/広域(DNS-SD)。**env** — 環境変数（インライン/置換）。**secrets** — SecretRef・プロバイダー設定。**auth** — 認証ストレージ・`auth.cooldowns`。
- **logging / diagnostics / update / acp / cli / wizard / identity / cron**（`cron.retry`/`failureAlert`/`failureDestination`）。**メディアモデルテンプレート変数**。**$include**。

## 設定・使い方の要点

- 編集前に `config.schema.lookup` で対象パスのスキーマ（フィールドの `title`/`description`・型・既定）を引く。Control UI もこのスキーマからフォームを描く。
- スキーマメタデータはネストオブジェクト・ワイルドカード（`*`）・配列アイテム（`[]`）・`anyOf`/`oneOf`/`allOf` まで引き継がれ、Plugin/チャネルのスキーマもマージされる。

## 注意点・落とし穴

- 本ページ（およびこの wiki 解説）は**スナップショット**。フィールドは増減するので、運用では必ずライブスキーマ（`openclaw config schema` / `config.schema.lookup`）を一次情報源にする。
- チャネル/Plugin 所有のコマンドカタログとメモリ/QMD の深い調整は、このリファレンスではなく各専用ページにある。

## 用語と略称

- **JSON Schema** = 設定の構造・型・制約を機械可読に定義する仕様
- **MCP** = Model Context Protocol（外部ツール/データ接続規格）
- **ACP** = Agent Client Protocol（外部エージェント制御）
- **mDNS / DNS-SD** = ローカル/広域のサービス検出

## 関連ページ

- [[concepts/configuration]] — 対応する概念ページ
- [[sources/gateway/configuration]] / [[sources/gateway/configuration-examples]]
- [[sources/gateway/config-agents]] / [[sources/gateway/config-channels]] / [[sources/gateway/config-tools]]
