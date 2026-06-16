---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/config-agents
source_path: raw/docs/gateway/config-agents.md
doc_section: gateway
title: "設定 — エージェント"
ingested: 2026-06-14
tags: [configuration, agents, session, messages, multi-agent, sandbox]
related:
  - "[[concepts/configuration]]"
  - "[[concepts/agent]]"
  - "[[concepts/multi-agent]]"
---

# 設定 — エージェント（解説）

> 原典: `raw/docs/gateway/config-agents.md` ・ https://docs.openclaw.ai/ja-JP/gateway/config-agents

## 一言まとめ

エージェントまわりの設定キー（`agents.*`・`multiAgent.*`・`session.*`・`messages.*`・`talk.*`）をフィールド単位で扱うリファレンス。ワークスペース・モデル・Heartbeat・Compaction・サンドボックス・マルチエージェントルーティング・セッション・メッセージ・TTS など。

## 位置づけ

[[concepts/configuration]] のエージェント・ドメイン。ここで設定する対象の概念は [[concepts/agent]]（1 プロセスの契約）・[[concepts/multi-agent]]（複数エージェント）・[[concepts/session]]・[[concepts/messages]] にそれぞれ対応する。チャネルは [[sources/gateway/config-channels]]、ツールは [[sources/gateway/config-tools]]。

## 仕組み・ふるまい（主なキー群）

- **`agents.defaults.*`**（全エージェント共通の既定）：`workspace`・`repoRoot`・`skills`（許可リスト）・`skipBootstrap`/`contextInjection`・`bootstrapMaxChars`/`bootstrapTotalMaxChars`（→ [[concepts/system-prompt]]）・`imageMaxDimensionPx`・`userTimezone`/`timeFormat`・**`model`**（`primary`＋`fallbacks`、`models` カタログ＝`/model` 許可リスト）・ランタイムポリシー（→ [[concepts/agent-runtimes]]）・`cliBackends`・`systemPromptOverride`/`promptOverlays`・**`heartbeat`**（`every`/`target`/`directPolicy`）・**`compaction`**（→ [[concepts/compaction]]）・`runRetries`・**`contextPruning`**（→ [[concepts/session-pruning]]）・ブロックストリーミング/入力インジケーター（→ [[concepts/streaming]]）・**`sandbox`**（`mode`: off/non-main/all、`scope`: session/agent/shared → [[concepts/sandboxing]]）。
- **`agents.list[]`**（エージェントごとの上書き）：`id`/`workspace`/`agentDir`/`model`/`skills`/`tools`/`sandbox`/`groupChat.mentionPatterns` 等。
- **マルチエージェントルーティング**：`agents.list` ＋ `bindings`（マッチフィールド：channel/accountId/peer/parentPeer/guildId+roles/teamId）＋エージェント単位アクセスプロファイル（→ [[concepts/multi-agent]]）。
- **`session.*`**：`dmScope`（main/per-peer/per-channel-peer/per-account-channel-peer）・`threadBindings`・`reset`（mode/atHour/idleMinutes）（→ [[concepts/session]]）。
- **`messages.*`**：応答接頭辞・確認リアクション・受信デバウンス・**TTS**（→ [[concepts/messages]]）。
- **`talk.*`**：音声対話モード。

## 設定・使い方の要点

- **Skills 制限**：`agents.defaults.skills`（共有ベースライン）→ `agents.list[].skills`（上書き、`[]` で無効、省略で継承、defaults 省略で無制限）。
- **モデルカタログの追記**は `config set agents.defaults.models '<json>' --strict-json --merge`（削除を伴う置換は `--replace` がないと拒否）。
- **サンドボックス**は先にイメージをビルド（`scripts/sandbox-setup.sh` か `docker build`）。

## 注意点・落とし穴

- `agents.list[].sandbox`/`tools` は Gateway レベルで強制され、人格ファイルとは独立（[[concepts/delegate-architecture]] の境界づけの要）。
- `tools.elevated` はグローバル・送信者ベースでエージェント単位にできない（境界が要るなら `agents.list[].tools` で `exec` を拒否）。
- 多くのキーが [[concepts/configuration]] のホットリロード対象（再起動不要）。

## 用語と略称

- **agents.defaults / agents.list** = 全エージェント既定 / エージェントごと上書き
- **dmScope** = DM をどの粒度で分離するか
- **bindings** = 受信メッセージを `agentId` に振り分ける規則
- **TTS** = Text-to-Speech（音声合成）

## 関連ページ

- [[concepts/configuration]] — 設定の全体像
- [[concepts/agent]] / [[concepts/multi-agent]] / [[concepts/session]] / [[concepts/messages]]
- [[sources/gateway/config-channels]] / [[sources/gateway/config-tools]] / [[sources/gateway/configuration-reference]]
