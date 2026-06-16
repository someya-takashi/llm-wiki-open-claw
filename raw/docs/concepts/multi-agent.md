---
title: "マルチエージェントルーティング"
source: "https://docs.openclaw.ai/ja-JP/concepts/multi-agent"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
1つの実行中の Gateway 内で、複数の\_分離された\_エージェントを実行できます。各エージェントは独自のワークスペース、状態ディレクトリ（ `agentDir` ）、セッション履歴を持ち、さらに複数のチャネルアカウント（例: 2つの WhatsApp）も併用できます。受信メッセージはバインディングを通じて適切なエージェントにルーティングされます。

ここでいう **エージェント** とは、ワークスペースファイル、認証プロファイル、モデルレジストリ、セッションストアを含む、ペルソナごとの完全なスコープです。 `agentDir` は、このエージェントごとの設定を `~/.openclaw/agents/<agentId>/` に保持するオンディスクの状態ディレクトリです。 **バインディング** は、チャネルアカウント（例: Slack ワークスペースや WhatsApp 番号）をそれらのエージェントの1つにマッピングします。

## 「1つのエージェント」とは何か？

**エージェント** は、独自の以下を持つ、完全にスコープ化された頭脳です。

- **ワークスペース** （ファイル、AGENTS.md/SOUL.md/USER.md、ローカルノート、ペルソナルール）。
- 認証プロファイル、モデルレジストリ、エージェントごとの設定のための **状態ディレクトリ** （ `agentDir` ）。
- `~/.openclaw/agents/<agentId>/sessions` 配下の **セッションストア** （チャット履歴 + ルーティング状態）。

認証プロファイルは **エージェントごと** です。各エージェントは独自の次の場所から読み取ります。

text

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

> [!note] Note
> **Note**
> 
> ここでも `sessions_history` は、より安全なクロスセッション想起パスです。生のトランスクリプトダンプではなく、境界付けされサニタイズされたビューを返します。アシスタントの想起では、考慮タグ、 `<relevant-memories>` の足場、プレーンテキストのツール呼び出し XML ペイロード（ `<tool_call>...</tool_call>` 、 `<function_call>...</function_call>` 、 `<tool_calls>...</tool_calls>` 、 `<function_calls>...</function_calls>` 、および切り詰められたツール呼び出しブロックを含む）、格下げされたツール呼び出しの足場、漏洩した ASCII/全角のモデル制御トークン、不正な MiniMax ツール呼び出し XML が、秘匿化/切り詰めの前に除去されます。

> [!note] Note
> **Warning**
> 
> 複数のエージェントで `agentDir` を再利用しないでください（認証/セッションの衝突を引き起こします）。エージェントはローカルプロファイルを持たない場合、デフォルト/メインエージェントの認証プロファイルを参照できますが、OpenClaw は OAuth リフレッシュトークンをセカンダリエージェントストアへ複製しません。独立した OAuth アカウントが必要な場合は、そのエージェントからサインインしてください。認証情報を手動でコピーする場合は、移植可能な静的 `api_key` または `token` プロファイルのみをコピーしてください。

Skills は各エージェントワークスペースと `~/.openclaw/skills` などの共有ルートから読み込まれ、設定されている場合は有効なエージェントの Skills 許可リストでフィルタリングされます。共有ベースラインには `agents.defaults.skills` を、エージェントごとの置き換えには `agents.list[].skills` を使用します。 [Skills: エージェントごと vs 共有](https://docs.openclaw.ai/ja-JP/tools/skills#per-agent-vs-shared-skills) と [Skills: エージェント Skills 許可リスト](https://docs.openclaw.ai/ja-JP/tools/skills#agent-skill-allowlists) を参照してください。

Gateway は **1つのエージェント** （デフォルト）または **多数のエージェント** を並べてホストできます。

> [!note] Note
> **Note**
> 
> **ワークスペースに関する注意:** 各エージェントのワークスペースは **デフォルト cwd** であり、強制的なサンドボックスではありません。相対パスはワークスペース内で解決されますが、サンドボックス化が有効でない限り、絶対パスはホスト上の他の場所に到達できます。 [サンドボックス化](https://docs.openclaw.ai/ja-JP/gateway/sandboxing) を参照してください。

## パス（クイックマップ）

- 設定: `~/.openclaw/openclaw.json` （または `OPENCLAW_CONFIG_PATH` ）
- 状態ディレクトリ: `~/.openclaw` （または `OPENCLAW_STATE_DIR` ）
- ワークスペース: `~/.openclaw/workspace` （または `~/.openclaw/workspace-<agentId>` ）
- エージェントディレクトリ: `~/.openclaw/agents/<agentId>/agent` （または `agents.list[].agentDir` ）
- セッション: `~/.openclaw/agents/<agentId>/sessions`

### 単一エージェントモード（デフォルト）

何もしない場合、OpenClaw は単一のエージェントを実行します。

- `agentId` のデフォルトは **`main`** です。
- セッションは `agent:main:<mainKey>` としてキー付けされます。
- ワークスペースのデフォルトは `~/.openclaw/workspace` です（ `OPENCLAW_PROFILE` が設定されている場合は `~/.openclaw/workspace-<profile>` ）。
- 状態のデフォルトは `~/.openclaw/agents/main/agent` です。

## エージェントヘルパー

新しい分離されたエージェントを追加するには、エージェントウィザードを使用します。

bash

```bash
openclaw agents add work
```

次に、受信メッセージをルーティングするために `bindings` を追加します（またはウィザードに任せます）。

次で確認します。

bash

```bash
openclaw agents list --bindings
```

## クイックスタート

- ### 各エージェントワークスペースを作成する
	ウィザードを使用するか、ワークスペースを手動で作成します。
	bash
	```bash
	openclaw agents add coding
	openclaw agents add social
	```
	各エージェントには、 `SOUL.md` 、 `AGENTS.md` 、任意の `USER.md` を含む独自のワークスペースに加え、専用の `agentDir` と `~/.openclaw/agents/<agentId>` 配下のセッションストアが割り当てられます。
- ### チャネルアカウントを作成する
	使用するチャネルごとに、エージェントごとのアカウントを作成します。
	- Discord: エージェントごとに1つのボットを作成し、Message Content Intent を有効にして、各トークンをコピーします。
	- Telegram: BotFather 経由でエージェントごとに1つのボットを作成し、各トークンをコピーします。
	- WhatsApp: アカウントごとに各電話番号をリンクします。
	bash
	```bash
	openclaw channels login --channel whatsapp --account work
	```
	チャネルガイドを参照してください: [Discord](https://docs.openclaw.ai/ja-JP/channels/discord) 、 [Telegram](https://docs.openclaw.ai/ja-JP/channels/telegram) 、 [WhatsApp](https://docs.openclaw.ai/ja-JP/channels/whatsapp) 。
- ### エージェント、アカウント、バインディングを追加する
	`agents.list` にエージェントを、 `channels.<channel>.accounts` にチャネルアカウントを追加し、 `bindings` でそれらを接続します（例は下記）。
- ### 再起動して確認する
	bash
	```bash
	openclaw gateway restart
	openclaw agents list --bindings
	openclaw channels status --probe
	```

## 複数エージェント = 複数の人、複数の人格

**複数エージェント** では、各 `agentId` が **完全に分離されたペルソナ** になります。

- **異なる電話番号/アカウント** （チャネルごとの `accountId` ）。
- **異なる人格** （ `AGENTS.md` や `SOUL.md` など、エージェントごとのワークスペースファイル）。
- **分離された認証 + セッション** （明示的に有効化しない限り相互干渉なし）。

これにより、 **複数の人** が1つの Gateway サーバーを共有しながら、それぞれの AI「頭脳」とデータを分離して維持できます。

## クロスエージェント QMD メモリ検索

あるエージェントが別のエージェントの QMD セッショントランスクリプトを検索する必要がある場合は、 `agents.list[].memorySearch.qmd.extraCollections` 配下に追加コレクションを追加します。すべてのエージェントが同じ共有トランスクリプトコレクションを継承する必要がある場合にのみ、 `agents.defaults.memorySearch.qmd.extraCollections` を使用します。

json5

```
{
  agents: {
    defaults: {
      workspace: "~/workspaces/main",
      memorySearch: {
        qmd: {
          extraCollections: [{ path: "~/agents/family/sessions", name: "family-sessions" }],
        },
      },
    },
    list: [
      {
        id: "main",
        workspace: "~/workspaces/main",
        memorySearch: {
          qmd: {
            extraCollections: [{ path: "notes" }], // resolves inside workspace -> collection named "notes-main"
          },
        },
      },
      { id: "family", workspace: "~/workspaces/family" },
    ],
  },
  memory: {
    backend: "qmd",
    qmd: { includeDefaultMemory: false },
  },
}
```

追加コレクションのパスはエージェント間で共有できますが、パスがエージェントワークスペースの外にある場合、コレクション名は明示されたままになります。ワークスペース内のパスはエージェントスコープのままなので、各エージェントは独自のトランスクリプト検索セットを保持します。

## 1つの WhatsApp 番号、複数の人（DM 分割）

**1つの WhatsApp アカウント** を使いながら、 **異なる WhatsApp DM** を別々のエージェントにルーティングできます。 `peer.kind: "direct"` で送信者 E.164（例: `+15551234567` ）にマッチさせます。返信は引き続き同じ WhatsApp 番号から送信されます（エージェントごとの送信者 ID はありません）。

> [!note] Note
> **Note**
> 
> ダイレクトチャットはエージェントの **メインセッションキー** に集約されるため、真の分離には **1人につき1エージェント** が必要です。

例:

json5

```
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

注意:

- DM アクセス制御は、エージェントごとではなく、 **WhatsApp アカウントごとにグローバル** です（ペアリング/許可リスト）。
- 共有グループの場合は、グループを1つのエージェントにバインドするか、 [ブロードキャストグループ](https://docs.openclaw.ai/ja-JP/channels/broadcast-groups) を使用します。

## ルーティングルール（メッセージがエージェントを選ぶ仕組み）

バインディングは **決定的** で、 **最も具体的なものが優先** されます。

- ### peer マッチ
	正確な DM/グループ/チャネル ID。
- ### parentPeer マッチ
	スレッド継承。
- ### guildId + roles
	Discord ロールルーティング。
- ### guildId
	Discord。
- ### teamId
	Slack。
- ### チャネルの accountId マッチ
	アカウントごとのフォールバック。
- ### チャネルレベルマッチ
	`accountId: "*"` 。
- ### デフォルトエージェント
	`agents.list[].default` にフォールバックし、それ以外は最初のリストエントリ、デフォルトは `main` 。

同順位の解決と AND セマンティクス
- 同じ階層で複数のバインディングがマッチした場合、設定順で最初のものが優先されます。
- バインディングが複数のマッチフィールド（例: `peer` + `guildId` ）を設定している場合、指定されたすべてのフィールドが必要です（ `AND` セマンティクス）。
アカウントスコープの詳細
- `accountId` を省略したバインディングは、デフォルトアカウントのみにマッチします。
- すべてのアカウントにまたがるチャネル全体のフォールバックには `accountId: "*"` を使用します。
- 後で同じエージェントに対して明示的なアカウント ID を持つ同じバインディングを追加した場合、OpenClaw は既存のチャネルのみのバインディングを複製せず、アカウントスコープにアップグレードします。

## 複数アカウント / 電話番号

**複数アカウント** をサポートするチャネル（例: WhatsApp）は、各ログインを識別するために `accountId` を使用します。各 `accountId` は異なるエージェントにルーティングできるため、1つのサーバーで複数の電話番号をホストしてもセッションが混在しません。

`accountId` が省略されたときのチャネル全体のデフォルトアカウントが必要な場合は、 `channels.<channel>.defaultAccount` を設定します（任意）。未設定の場合、OpenClaw は存在すれば `default` にフォールバックし、それ以外は設定済みアカウント ID の先頭（ソート済み）にフォールバックします。

このパターンをサポートする一般的なチャネルには、次のものがあります。

- `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`
- `irc`, `line`, `googlechat`, `mattermost`, `matrix`, `nextcloud-talk`
- `zalo`, `zalouser`, `nostr`, `feishu`

## 概念

- `agentId`: 1つの「頭脳」（ワークスペース、エージェントごとの認証、エージェントごとのセッションストア）。
- `accountId`: 1つのチャネルアカウントインスタンス（例: WhatsApp アカウント `"personal"` と `"biz"` ）。
- `binding`: `(channel, accountId, peer)` と任意の guild/team ID によって、受信メッセージを `agentId` にルーティングします。
- ダイレクトチャットは `agent:<agentId>:<mainKey>` に集約されます（エージェントごとの「main」、 `session.mainKey` ）。

## プラットフォーム例

エージェントごとの Discord ボット

各 Discord ボットアカウントは一意の `accountId` にマッピングされます。各アカウントをエージェントにバインドし、ボットごとに許可リストを維持します。

json5

```
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```
- 各ボットをギルドに招待し、メッセージコンテンツインテントを有効にします。
- トークンは `channels.discord.accounts.<id>.token` に置きます（デフォルトアカウントでは `DISCORD_BOT_TOKEN` を使用できます）。
エージェントごとの Telegram ボット

json5

```
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```
- BotFather でエージェントごとに1つのボットを作成し、それぞれのトークンをコピーします。
- トークンは `channels.telegram.accounts.<id>.botToken` に置きます（デフォルトアカウントでは `TELEGRAM_BOT_TOKEN` を使用できます）。
エージェントごとの WhatsApp 番号

Gateway を起動する前に各アカウントをリンクします。

bash

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5):

js

```js
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },
 
  // Deterministic routing: first match wins (most-specific first).
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
 
    // Optional per-peer override (example: send a specific group to work agent).
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],
 
  // Off by default: agent-to-agent messaging must be explicitly enabled + allowlisted.
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },
 
  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // Optional override. Default: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## 一般的なパターン

### WhatsApp の日常利用 + Telegram での集中作業

チャンネルごとに分割します。WhatsApp は高速な日常用エージェントに、Telegram は Opus エージェントにルーティングします。

json5

```
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-6",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

注記:

- 1つのチャンネルに複数のアカウントがある場合は、バインディングに `accountId` を追加します（例: `{ channel: "whatsapp", accountId: "personal" }` ）。
- ほかはチャットのまま、単一の DM/グループを Opus にルーティングするには、そのピア用の `match.peer` バインディングを追加します。ピア一致は常にチャンネル全体のルールより優先されます。

### 同じチャンネルで、1つのピアだけ Opus へ

WhatsApp は高速エージェントのままにし、1つの DM だけを Opus にルーティングします。

json5

```
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-6",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

ピアバインディングは常に優先されるため、チャンネル全体のルールより上に置いてください。

### WhatsApp グループにバインドされたファミリーエージェント

専用のファミリーエージェントを単一の WhatsApp グループにバインドし、メンションによるゲーティングとより厳密なツールポリシーを設定します。

json5

```
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

注記:

- ツールの許可/拒否リストは **ツール** であり、Skills ではありません。Skill がバイナリを実行する必要がある場合は、 `exec` が許可され、そのバイナリがサンドボックス内に存在することを確認してください。
- より厳密にゲーティングするには、 `agents.list[].groupChat.mentionPatterns` を設定し、そのチャンネルのグループ許可リストを有効なままにします。

## エージェントごとのサンドボックスとツール設定

各エージェントは、独自のサンドボックスとツール制限を持つことができます。

js

```js
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // No sandbox for personal agent
        },
        // No tool restrictions - all tools available
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // Always sandboxed
          scope: "agent",  // One container per agent
          docker: {
            // Optional one-time setup after container creation
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // Only read tool
          deny: ["exec", "write", "edit", "apply_patch"],    // Deny others
        },
      },
    ],
  },
}
```

> [!note] Note
> **Note**
> 
> `setupCommand` は `sandbox.docker` の下にあり、コンテナ作成時に1回実行されます。解決後のスコープが `"shared"` の場合、エージェントごとの `sandbox.docker.*` オーバーライドは無視されます。

**利点:**

- **セキュリティ分離**: 信頼できないエージェントのツールを制限します。
- **リソース制御**: 特定のエージェントをサンドボックス化し、ほかのエージェントはホスト上に残します。
- **柔軟なポリシー**: エージェントごとに異なる権限を設定できます。

> [!note] Note
> **Note**
> 
> `tools.elevated` は **グローバル** かつ送信者ベースです。エージェントごとには設定できません。エージェントごとの境界が必要な場合は、 `agents.list[].tools` を使用して `exec` を拒否します。グループを対象にする場合は、 `agents.list[].groupChat.mentionPatterns` を使用して、@メンションが意図したエージェントに明確に対応するようにします。

詳しい例については、 [マルチエージェントのサンドボックスとツール](https://docs.openclaw.ai/ja-JP/tools/multi-agent-sandbox-tools) を参照してください。

## 関連

- [ACP エージェント](https://docs.openclaw.ai/ja-JP/tools/acp-agents) — 外部コーディングハーネスを実行する
- [チャンネルルーティング](https://docs.openclaw.ai/ja-JP/channels/channel-routing) — メッセージがエージェントにルーティングされる仕組み
- [プレゼンス](https://docs.openclaw.ai/ja-JP/concepts/presence) — エージェントのプレゼンスと可用性
- [セッション](https://docs.openclaw.ai/ja-JP/concepts/session) — セッションの分離とルーティング
- [サブエージェント](https://docs.openclaw.ai/ja-JP/tools/subagents) — バックグラウンドのエージェント実行を生成する