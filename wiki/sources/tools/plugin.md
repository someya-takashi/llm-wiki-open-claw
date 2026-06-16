---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/plugin
source_path: raw/docs/tools/plugin.md
doc_section: tools
title: "Plugin（インストール・設定・管理）"
ingested: 2026-06-14
tags: [plugin, plugin-system, install, slots, registry, native, bundle]
related:
  - "[[components/plugin-system]]"
  - "[[components/clawhub]]"
  - "[[concepts/configuration]]"
---

# Plugin（インストール・設定・管理）（解説）

> 原典: `raw/docs/tools/plugin.md` ・ https://docs.openclaw.ai/ja-JP/tools/plugin

## 一言まとめ

Plugin は OpenClaw を新機能で拡張する仕組み——チャネル・モデルプロバイダー・エージェントハーネス・ツール・Skills・音声・メディア理解/生成・Web 取得/検索など。**コア（同梱）**と**外部**があり、外部の多くは [[components/clawhub]] で配布される。

## 位置づけ

[[components/plugin-system]] の中核ソース。インストール元（ClawHub/npm/git/ローカル）と検出・有効化・スロット・ランタイム登録（`register(api)`）を定義する。設定は [[concepts/configuration]] の `plugins.*`。

## 仕組み・ふるまい

- **2 形式**：**ネイティブ**（`openclaw.plugin.json`＋ランタイムモジュール、プロセス内実行、任意機能を登録）と**バンドル**（Codex/Claude/Cursor 互換レイアウトを機能マッピング、より狭い信頼境界。詳細 [[sources/plugins/bundles]]）。
- **検出順（先勝ち）**：`plugins.load.paths` → ワークスペース Plugin（既定無効）→ グローバル Plugin → バンドル Plugin（多くは既定有効）。
- **登録 API**：`register(api)` で `registerProvider`/`registerChannel`/`registerTool`/`registerHook`/`registerSpeechProvider`/`registerRealtimeVoiceProvider`/`registerMediaUnderstandingProvider`/`registerVideoGenerationProvider`/`registerHttpRoute`/`registerCli`/`registerService` 等。`registrationMode`（full/discovery/setup-only/cli-metadata）でライブ副作用をガード。
- **スロット（排他カテゴリ）**：`plugins.slots.memory`（既定 `memory-core`）・`contextEngine`（既定 `legacy`）。一度に 1 つだけアクティブ。

## 設定・使い方の要点

- CLI：`openclaw plugins list|search|install|update|uninstall|inspect|enable|disable|registry`。インストール元指定 `clawhub:`/`npm:`/`git:`/ローカルパス（裸指定は ClawHub→npm）。チャットネイティブは `/plugins enable|disable`（プロセス内リロード）・`/plugins install`（要再起動）。
- `plugins.*`：`enabled`（マスタ）・`allow`（排他許可リスト）・`deny`（優先）・`bundledDiscovery`（既定 `allowlist`）・`entries.<id>.{enabled,config}`・`load.paths`・`slots`。会話フック（`before_agent_reply` 等）は `entries.<id>.hooks.allowConversationAccess=true` が必要。

## 注意点・落とし穴

- ⚠️ 無効な Plugin 設定はフェイルクローズ（`openclaw doctor --fix` で隔離）。所有権ブロック（`uid` 不一致）はファイル所有を修正。`/plugins enable` はプロセス内リロード、インストール/更新/アンインストールは Gateway 再起動が必要（ラッパーコンテナでは子 `gateway run` を再起動）。
- 重複チャネル/ツール所有権は `channelConfigs.<id>.preferOver` か一方を無効化で解消。`--dangerously-force-unsafe-install` は危険スキャナーの誤検知用の非常口。

## 用語と略称

- **Plugin** = OpenClaw のランタイム拡張
- **ネイティブ / バンドル** = プロセス内実行の本格拡張 / コンテンツパックの機能マッピング
- **スロット（slot）** = 排他的に 1 つだけ選ぶ Plugin カテゴリ（memory 等）
- **register(api)** = Plugin が機能を登録するエントリ
- **MCP** = Model Context Protocol（外部ツールを公開する規格）
- **ClawHub** = OpenClaw の Plugin/Skill マーケットプレイス

## 関連ページ

- [[components/plugin-system]] — 対応する構成要素ページ
- [[sources/plugins/manage-plugins]] / [[sources/plugins/bundles]] / [[sources/plugins/community]]
- [[components/clawhub]] / [[concepts/configuration]] / [[concepts/sandboxing]]
