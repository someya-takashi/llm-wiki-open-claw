---
type: concept
aliases: [Slash Commands, スラッシュコマンド, ディレクティブ, Directives]
tags: [slash-commands, directives, commands, owner-only, native-commands]
related:
  - "[[concepts/messages]]"
  - "[[concepts/session]]"
  - "[[concepts/channel-docking]]"
  - "[[concepts/skills]]"
components:
  - "[[components/cli]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/tools/slash-commands]]"
updated: 2026-06-14
---

# スラッシュコマンド（Slash Commands）

スラッシュコマンド（slash commands, `/...` で始まる単独メッセージで Gateway を直接操作する仕組み）は、自然言語のチャットと並ぶ**明示的な制御面**。`/new`・`/status`・`/think`・`/model` などを、あらゆるサーフェス（[[components/cli]]・[[components/webchat]]・[[components/control-ui]]・各チャネル）から同じように使える。

## 3 つの系統

| 系統 | 例 | ふるまい |
|---|---|---|
| **コマンド** | `/new`・`/compact`・`/export-session`・`/skill` | 単独 `/...` で実行 |
| **ディレクティブ** | `/think`・`/fast`・`/model`・`/queue`・`/elevated`・`/exec` | モデルに見える前に除去。**単独なら [[concepts/session]] に永続化**、混在ならインラインヒント |
| **インラインショートカット** | `/help`・`/status`・`/whoami` | 通常メッセージ内でも即時実行（承認済み送信者のみ） |

ディレクティブとショートカットは [[concepts/messages]] の受信処理でテキストから剥がされ、残りが通常フローへ流れる。**承認済み送信者にのみ**適用され（`commands.allowFrom`/チャネル許可リスト/`useAccessGroups`）、未承認者にはプレーンテキスト扱い。

## なぜ重要か

スラッシュコマンドは「モデルに頼らず確実に効く」操作面で、思考レベル・モデル・キュー戦略・サンドボックスといったセッション設定を**その場で・決定論的に**切り替えられる。許可リスト送信者のコマンドのみメッセージは**キュー＋モデルをバイパスし、グループのメンションゲートも越える**ため、運用の即応性を担保する。一方で `/config`・`/mcp`・`/plugins`・`/debug`・`/restart` のような危険な書き込みは**オーナー専用**（`commands.<x>: true`＋`ownerAllowFrom`）に隔離され、[[concepts/security]] の最小権限と整合する。

## 既存 wiki とのつながり

- **ネイティブ vs テキスト**：`commands.native` は Discord/Telegram にスラッシュコマンドを登録（分離セッション）、`commands.text` はチャット内 `/...` 解析（ネイティブ非対応サーフェスでも動く）。
- **生成コマンド**：[[concepts/channel-docking]] の dock コマンド（`/dock-discord` 等）、[[concepts/skills]] の `user-invocable` Skill コマンド（`/skill <name>`・直接 `/prose` 等）、バンドル Plugin コマンド（`/dreaming`・`/pair`・`/voice`・`/codex`）は [[components/plugin-system]] が供給。
- ⚠️ `/reasoning`・`/verbose`・`/trace` はグループで内部 reasoning/ツール出力/Plugin 診断を晒すリスク（[[concepts/threat-model]]）。

## 代表ソース

- [[sources/tools/slash-commands]] — 系統・設定・全コマンド一覧・サーフェス別挙動

## 関連ページ

- [[concepts/messages]] / [[concepts/session]] / [[concepts/channel-docking]] / [[concepts/skills]]
- [[concepts/queue]] / [[concepts/compaction]]
- [[components/cli]] / [[components/control-ui]] / [[components/gateway]]
