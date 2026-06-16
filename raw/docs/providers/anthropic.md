---
title: "Anthropic"
source: "https://docs.openclaw.ai/ja-JP/providers/anthropic"
author:
published:
created: 2026-06-14
description: "OpenClawでAPIキーまたはClaude CLI経由でAnthropic Claudeを使用する"
tags:
  - "clippings"
---
Anthropic は **Claude** モデルファミリーを構築しています。OpenClaw は 2 つの認証ルートをサポートします。

- **API key** — 使用量ベースの課金による Anthropic API への直接アクセス（ `anthropic/*` モデル）
- **Claude CLI** — 同じホスト上の既存の Claude CLI ログインを再利用

> [!note] Note
> **Warning**
> 
> Anthropic スタッフは、OpenClaw スタイルの Claude CLI 利用が再び許可されたと私たちに伝えたため、 Anthropic が新しいポリシーを公開しない限り、OpenClaw は Claude CLI の再利用と `claude -p` の利用を認可済みとして扱います。
> 
> 長期稼働する Gateway ホストでは、Anthropic API key が依然として最も明確で 予測しやすい本番パスです。
> 
> Anthropic の現在の公開ドキュメント:
> 
> - [Claude Code CLI リファレンス](https://code.claude.com/docs/en/cli-reference)
> - [Claude Agent SDK 概要](https://platform.claude.com/docs/en/agent-sdk/overview)
> - [Pro または Max プランで Claude Code を使う](https://support.claude.com/en/articles/11145838-using-claude-code-with-your-pro-or-max-plan)
> - [Team または Enterprise プランで Claude Code を使う](https://support.anthropic.com/en/articles/11845131-using-claude-code-with-your-team-or-enterprise-plan/)

## はじめに

### API key

**最適な用途:** 標準的な API アクセスと使用量ベースの課金。

- ### API key を取得する
	[Anthropic Console](https://console.anthropic.com/) で API key を作成します。
- ### オンボーディングを実行する
	bash
	```bash
	openclaw onboard
	# choose: Anthropic API key
	```
	または、キーを直接渡します。
	bash
	```bash
	openclaw onboard --anthropic-api-key "$ANTHROPIC_API_KEY"
	```
- ### モデルが利用可能であることを確認する
	bash
	```bash
	openclaw models list --provider anthropic
	```

### 設定例

json5

```
{
  env: { ANTHROPIC_API_KEY: "sk-ant-..." },
  agents: { defaults: { model: { primary: "anthropic/claude-opus-4-6" } } },
}
```

### Claude CLI

**最適な用途:** 別の API key なしで既存の Claude CLI ログインを再利用する。

- ### Claude CLI がインストール済みでログイン済みであることを確認する
	次で確認します。
	bash
	```bash
	claude --version
	```
- ### オンボーディングを実行する
	bash
	```bash
	openclaw onboard
	# choose: Claude CLI
	```
	OpenClaw は既存の Claude CLI 認証情報を検出して再利用します。
- ### モデルが利用可能であることを確認する
	bash
	```bash
	openclaw models list --provider anthropic
	```

> [!note] Note
> **Note**
> 
> Claude CLI バックエンドのセットアップとランタイム詳細は [CLI バックエンド](https://docs.openclaw.ai/ja-JP/gateway/cli-backends) にあります。

### 設定例

正規の Anthropic モデル参照に CLI ランタイムのオーバーライドを組み合わせることを推奨します。

json5

```
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-7" },
      models: {
        "anthropic/claude-opus-4-7": {
          agentRuntime: { id: "claude-cli" },
        },
      },
    },
  },
}
```

互換性のため、従来の `claude-cli/claude-opus-4-7` モデル参照も引き続き機能しますが、 新しい設定では provider/model 選択を `anthropic/*` のままにし、実行バックエンドは provider/model ランタイムポリシーに置くべきです。

> [!note] Note
> **Tip**
> 
> 最も明確な課金パスが必要な場合は、代わりに Anthropic API key を使用してください。OpenClaw は [OpenAI Codex](https://docs.openclaw.ai/ja-JP/providers/openai) 、 [Qwen Cloud](https://docs.openclaw.ai/ja-JP/providers/qwen) 、 [MiniMax](https://docs.openclaw.ai/ja-JP/providers/minimax) 、 [Z.AI / GLM](https://docs.openclaw.ai/ja-JP/providers/glm) のサブスクリプション形式のオプションもサポートします。

## thinking のデフォルト（Claude 4.6）

Claude 4.6 モデルは、明示的な thinking レベルが設定されていない場合、OpenClaw でデフォルトで `adaptive` thinking になります。

メッセージごとに `/think:<level>` で、またはモデルパラメータでオーバーライドします。

json5

```
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { thinking: "adaptive" },
        },
      },
    },
  },
}
```

> [!note] Note
> **Note**
> 
> 関連する Anthropic ドキュメント:
> 
> - [Adaptive thinking](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
> - [Extended thinking](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)

## プロンプトキャッシュ

OpenClaw は、API-key 認証向けに Anthropic のプロンプトキャッシュ機能をサポートします。

| 値 | キャッシュ期間 | 説明 |
| --- | --- | --- |
| `"short"` (デフォルト) | 5 分 | API-key 認証で自動的に適用されます |
| `"long"` | 1 時間 | 拡張キャッシュ |
| `"none"` | キャッシュなし | プロンプトキャッシュを無効化 |

json5

```
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
  },
}
```

エージェントごとのキャッシュオーバーライド

モデルレベルのパラメータを基準として使用し、その後 `agents.list[].params` で特定のエージェントをオーバーライドします。

json5

```
{
  agents: {
    defaults: {
      model: { primary: "anthropic/claude-opus-4-6" },
      models: {
        "anthropic/claude-opus-4-6": {
          params: { cacheRetention: "long" },
        },
      },
    },
    list: [
      { id: "research", default: true },
      { id: "alerts", params: { cacheRetention: "none" } },
    ],
  },
}
```

設定のマージ順序:

1. `agents.defaults.models["provider/model"].params`
2. `agents.list[].params` （ `id` が一致するもの。キーごとにオーバーライド）

これにより、あるエージェントは長期間有効なキャッシュを保持し、同じモデル上の別のエージェントはバースト的または再利用の少ないトラフィック向けにキャッシュを無効化できます。

Bedrock Claude の注記
- Bedrock 上の Anthropic Claude モデル（ `amazon-bedrock/*anthropic.claude*` ）は、設定されている場合 `cacheRetention` のパススルーを受け入れます。
- Anthropic 以外の Bedrock モデルは、ランタイムで `cacheRetention: "none"` に強制されます。
- API-key のスマートデフォルトは、明示的な値が設定されていない場合、Claude-on-Bedrock 参照にも `cacheRetention: "short"` をシードします。

## 高度な設定

高速モード

OpenClaw の共有 `/fast` トグルは、Anthropic への直接トラフィック（API-key と `api.anthropic.com` への OAuth）をサポートします。

| コマンド | 対応先 |
| --- | --- |
| `/fast on` | `service_tier: "auto"` |
| `/fast off` | `service_tier: "standard_only"` |

json5

```
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-sonnet-4-6": {
          params: { fastMode: true },
        },
      },
    },
  },
}
```

> [!note] Note
> **Note**
> - 直接の `api.anthropic.com` リクエストにのみ注入されます。プロキシルートでは `service_tier` は変更されません。
> - 明示的な `serviceTier` または `service_tier` パラメータは、両方が設定されている場合 `/fast` をオーバーライドします。
> - Priority Tier 容量がないアカウントでは、 `service_tier: "auto"` が `standard` に解決される場合があります。

メディア理解（画像と PDF）

同梱の Anthropic plugin は、画像と PDF の理解を登録します。OpenClaw は 設定された Anthropic 認証からメディア機能を自動解決します。追加の 設定は不要です。

| プロパティ | 値 |
| --- | --- |
| デフォルトモデル | `claude-opus-4-7` |
| サポート入力 | 画像、PDF ドキュメント |

画像または PDF が会話に添付されると、OpenClaw は自動的に Anthropic メディア理解プロバイダー経由でルーティングします。

1M コンテキストウィンドウ（ベータ）

Anthropic の 1M コンテキストウィンドウはベータゲート付きです。モデルごとに有効化します。

json5

```
{
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": {
          params: { context1m: true },
        },
      },
    },
  },
}
```

OpenClaw はリクエスト上でこれを `anthropic-beta: context-1m-2025-08-07` にマップします。

`params.context1m: true` は、対象となる Opus および Sonnet モデルの Claude CLI バックエンド （ `claude-cli/*` ）にも適用され、それらの CLI セッションのランタイム コンテキストウィンドウを直接 API の挙動に合わせて拡張します。

> [!note] Note
> **Warning**
> 
> Anthropic 認証情報で長いコンテキストへのアクセスが必要です。従来のトークン認証（ `sk-ant-oat-*` ）は 1M コンテキストリクエストでは拒否されます。OpenClaw は警告をログに記録し、標準のコンテキストウィンドウにフォールバックします。

Claude Opus 4.7 1M コンテキスト

`anthropic/claude-opus-4.7` とその `claude-cli` バリアントは、デフォルトで 1M コンテキスト ウィンドウを持ちます。 `params.context1m: true` は不要です。

## トラブルシューティング

401 エラー / トークンが突然無効になった

Anthropic トークン認証は期限切れになり、取り消される場合があります。新しいセットアップでは、代わりに Anthropic API key を使用してください。

provider "anthropic" の API key が見つかりません

Anthropic 認証は **エージェントごと** です。新しいエージェントはメインエージェントのキーを継承しません。そのエージェントでオンボーディングを再実行するか（または Gateway ホストで API key を設定し）、その後 `openclaw models status` で確認してください。

profile "anthropic:default" の認証情報が見つかりません

`openclaw models status` を実行して、どの認証プロファイルがアクティブかを確認してください。オンボーディングを再実行するか、そのプロファイルパスに API key を設定してください。

利用可能な認証プロファイルがありません（すべてクールダウン中）

`openclaw models status --json` で `auth.unusableProfiles` を確認してください。Anthropic のレート制限クールダウンはモデルスコープの場合があるため、同系統の別の Anthropic モデルはまだ利用できる場合があります。別の Anthropic プロファイルを追加するか、クールダウンを待ってください。

> [!note] Note
> **Note**
> 
> 追加のヘルプ: [トラブルシューティング](https://docs.openclaw.ai/ja-JP/help/troubleshooting) と [FAQ](https://docs.openclaw.ai/ja-JP/help/faq) 。

## 関連[**モデル選択**

プロバイダー、モデル参照、フェイルオーバー挙動の選択。

](https://docs.openclaw.ai/ja-JP/concepts/model-providers)

[

**CLI バックエンド**

Claude CLI バックエンドのセットアップとランタイム詳細。

](https://docs.openclaw.ai/ja-JP/gateway/cli-backends)[

**プロンプトキャッシュ**

プロバイダー全体でプロンプトキャッシュがどのように機能するか。

](https://docs.openclaw.ai/ja-JP/reference/prompt-caching)[

**OAuth と認証**

認証の詳細と認証情報の再利用ルール。

](https://docs.openclaw.ai/ja-JP/gateway/authentication)