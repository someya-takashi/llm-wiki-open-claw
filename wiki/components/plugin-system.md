---
type: component
aliases: [Plugin System, プラグインシステム, Plugins, Plugin]
tags: [plugin-system, plugin, extension, registry, slot, bundle]
concepts:
  - "[[concepts/architecture]]"
  - "[[concepts/configuration]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/sandboxing]]"
related:
  - "[[components/gateway]]"
  - "[[components/clawhub]]"
sources:
  - "[[sources/tools/plugin]]"
  - "[[sources/plugins/manage-plugins]]"
  - "[[sources/plugins/bundles]]"
  - "[[sources/plugins/community]]"
  - "[[sources/plugins/building-plugins]]"
  - "[[sources/plugins/hooks]]"
updated: 2026-06-14
---

# Plugin System（プラグインシステム）

**Plugin System** は OpenClaw の**拡張機構**——チャネル・モデルプロバイダー・エージェントハーネス・ツール・Skills・音声・メディア理解/生成・Web 取得/検索などを後付けで足せる仕組み。多くの組み込み機能自体が「コア Plugin」として実装され、外部 Plugin は [[components/clawhub]]・npm・git・ローカルから入る。[[components/gateway]] のプロセス内で登録・実行され、全体像は [[concepts/architecture]] の一部。

## 2 つの形式

| 形式 | 仕組み | 信頼境界 |
|---|---|---|
| **ネイティブ** | `openclaw.plugin.json`＋ランタイムモジュール。`register(api)` で任意機能を登録、プロセス内実行 | 広い（任意コード） |
| **バンドル** | Codex/Claude/Cursor 互換レイアウトを Skills/フック/MCP/LSP に機能マッピング | 狭い（コンテンツパック、[[sources/plugins/bundles]]） |

ネイティブの登録 API は `registerProvider`/`registerChannel`/`registerTool`/`registerHook`/`registerSpeechProvider`/`registerRealtimeVoiceProvider`/`registerMediaUnderstandingProvider`/`registerVideoGenerationProvider`/`registerHttpRoute`/`registerCli`/`registerService` 等。`registrationMode`（full/discovery/setup-only/cli-metadata）でライブ副作用をガードする。

## 仕組み

- **検出順（先勝ち）**：`plugins.load.paths` → ワークスペース Plugin（既定無効）→ グローバル Plugin → バンドル Plugin（多くは既定有効）。
- **設定**（[[concepts/configuration]] の `plugins.*`）：`enabled`（マスタ）・`allow`（排他許可リスト）・`deny`（優先）・`bundledDiscovery`（既定 `allowlist`）・`entries.<id>.{enabled,config}`・`slots`。
- **スロット（排他カテゴリ）**：一度に 1 つだけ。`memory`（既定 `memory-core`、他に [[sources/plugins/memory-lancedb]]）・`contextEngine`（既定 `legacy`、[[concepts/context-engine]]）。
- **ライフサイクル**：`/plugins enable|disable` はプロセス内リロード、`install`/`update`/`uninstall` は Gateway 再起動が必要。永続レジストリ（`plugins/installs.json`）をコールドリードに使う。

## なぜ重要か

「コアを小さく保ちつつ、必要な統合だけを足す」設計の要。チャネル（Slack/WhatsApp/Zalo…）もモデルプロバイダー（Anthropic/OpenAI…）も実体は Plugin であり、Plugin System が OpenClaw の拡張性とエコシステム（[[components/clawhub]]）を支える。バンドルの狭い信頼境界やインストールスキャンは [[concepts/threat-model]] のサプライチェーン対策と直結する。

## 代表的な Plugin（本 wiki 収録）

- ハーネス：[[sources/plugins/codex-harness]]（[[concepts/agent-runtimes]]）
- メモリ：[[sources/plugins/memory-lancedb]] / [[sources/plugins/memory-wiki]]（[[concepts/memory]]）
- 音声：[[sources/plugins/voice-call]] / [[sources/plugins/google-meet]]（[[concepts/voice]]）
- 入口/ユーティリティ：[[sources/plugins/webhooks]] / [[sources/plugins/oc-path]]
- ワークフロー：[[sources/prose]]（OpenProse、`.prose` マルチエージェント DSL）
- チャネル：[[channels/zalouser]]

## Plugin SDK（作成者向け）

ネイティブ Plugin の作り方は **[[sources/plugins/building-plugins]]**（入口）を起点に、種類別ガイド——チャネル [[sources/plugins/sdk-channel-plugins]]、プロバイダー [[sources/plugins/sdk-provider-plugins]]、CLI バックエンド [[sources/plugins/cli-backend-plugins]]——と、ライフサイクル介入の **[[sources/plugins/hooks]]**（`before_tool_call`/`llm_input` 等）。コアに新ドメイン（capability）を足すコントリビューターは [[sources/plugins/adding-capabilities]]。多くの組み込みプロバイダー（[[concepts/agent-runtimes]]）・チャネル（[[concepts/messages]]）・Skill（[[concepts/skills]]）が同じ SDK 面で実装される。

## 関連

- [[concepts/architecture]] / [[concepts/configuration]] / [[concepts/agent-runtimes]] / [[concepts/sandboxing]] / [[concepts/threat-model]] / [[concepts/skills]]
- [[components/gateway]] / [[components/clawhub]]
- [[sources/tools/plugin]] / [[sources/tools/tools]] / [[sources/plugins/manage-plugins]] / [[sources/plugins/bundles]] / [[sources/plugins/building-plugins]]
