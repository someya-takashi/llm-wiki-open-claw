---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/slash-commands
source_path: raw/docs/tools/slash-commands.md
doc_section: tools
title: "スラッシュコマンド"
ingested: 2026-06-14
tags: [slash-commands, directives, commands, owner-only, native, text]
related:
  - "[[concepts/slash-commands]]"
  - "[[concepts/messages]]"
  - "[[components/cli]]"
---

# スラッシュコマンド（解説）

> 原典: `raw/docs/tools/slash-commands.md` ・ https://docs.openclaw.ai/ja-JP/tools/slash-commands

## 一言まとめ

`/...` で始まる**単独メッセージ**で Gateway を直接操作する仕組み。コマンド（`/new`/`/status`…）・ディレクティブ（`/think`/`/model`/`/queue`…セッション設定を変える）・インラインショートカット（承認済み送信者のみ即時実行）の 3 系統がある。

## 位置づけ

[[concepts/slash-commands]] の中核ソース。解析は [[concepts/messages]] の受信パイプライン、適用先はセッション（[[concepts/session]]）。すべてのサーフェス（[[components/cli]]・[[components/webchat]]・[[components/control-ui]]・各チャネル）で働く。

## 仕組み・ふるまい

- **コマンド vs ディレクティブ**：ディレクティブはモデルに見える前に除去。**ディレクティブのみ**のメッセージはセッションに**永続化**＋確認応答、通常チャット内の混在は「インラインヒント」で永続化されない。承認済み送信者にのみ適用（`commands.allowFrom`/チャネル許可リスト/`useAccessGroups`）。
- **ネイティブ vs テキスト**：`commands.native`（Discord/Telegram は自動オン、Slack はオフ）で各チャネルにネイティブコマンド登録。`commands.text` でチャット内 `/...` 解析（ネイティブ非対応サーフェスでも動く）。ネイティブコマンドは分離セッション（`agent:<id>:discord:slash:<user>` 等）。
- **オーナー専用**：`/config`・`/mcp`・`/plugins`（書き込み）・`/debug`・`/diagnostics`・`/restart` は `commands.<x>: true`＋`ownerAllowFrom` が必要。`enforceOwnerForCommands` でチャネルごとにオーナー ID を要求。

## 設定・使い方の要点

- 主要コマンド：セッション（`/new`・`/reset [soft]`・`/compact`・`/stop`・`/export-session`/`/export-trajectory`）、実行制御（`/think`・`/fast`・`/verbose`・`/trace`・`/reasoning`・`/elevated`・`/exec`・`/model`・`/models`・`/queue`・`/steer`）、検出（`/help`・`/tools`・`/status`・`/context`・`/whoami`・`/usage`）、Skill/承認（`/skill`・`/allowlist`・`/approve`・`/btw`）、サブエージェント/ACP（`/subagents`・`/acp`・`/focus`）。
- **dock コマンド**（`/dock-discord` 等）は返信ルートを別チャネルへ（[[concepts/channel-docking]]、`session.identityLinks` 必須）。**Skill コマンド**は `user-invocable` skill が `/skill <name>` や直接 `/prose` 等として公開（[[concepts/skills]]）。バンドル Plugin コマンド（`/dreaming`・`/pair`・`/voice`・`/codex` 等）。
- 高速パス：許可リスト送信者のコマンドのみメッセージはキュー＋モデルをバイパスし、グループのメンションゲートも越える。

## 注意点・落とし穴

- ⚠️ `/reasoning`・`/verbose`・`/trace` は**グループで内部 reasoning/ツール出力/Plugin 診断を晒すリスク**（グループでは off 推奨）。Slack は `/status` 予約のため `/agentstatus` を登録。`/config`/`/debug` は別物（ディスク書き込み vs ランタイムのみ上書き）。

## 用語と略称

- **コマンド / ディレクティブ** = 単独 `/...` で動く操作 / セッション設定を変える `/...`
- **インラインショートカット** = 通常メッセージ内でも即時実行される一部コマンド（`/status` 等）
- **ネイティブコマンド** = Discord/Telegram/Slack が UI に登録するスラッシュコマンド
- **dock コマンド** = 返信ルートを別チャネルへ切り替える（[[concepts/channel-docking]]）
- **オーナー専用（owner-only）** = オペレーター ID を要する危険コマンド

## 関連ページ

- [[concepts/slash-commands]] — 対応する概念ページ
- [[concepts/messages]] / [[concepts/session]] / [[concepts/channel-docking]] / [[concepts/skills]]
- [[components/cli]] / [[components/control-ui]]
