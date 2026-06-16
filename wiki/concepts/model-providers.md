---
type: concept
aliases: [Model Providers, モデルプロバイダー, LLM Providers, models]
tags: [model-providers, llm, provider, model-reference, catalog, auth]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/model-failover]]"
  - "[[concepts/oauth]]"
  - "[[concepts/local-models]]"
components:
  - "[[components/gateway]]"
  - "[[components/plugin-system]]"
sources:
  - "[[sources/concepts/model-providers]]"
  - "[[sources/concepts/models]]"
  - "[[sources/providers/providers]]"
  - "[[sources/providers/models]]"
updated: 2026-06-14
---

# モデルプロバイダー（Model Providers）

モデルプロバイダー（model providers, LLM を提供する事業者/エンドポイント）は、エージェントの「頭脳」を供給する層。モデルは **`provider/model`** 参照（例 `anthropic/claude-opus-4-6`・`openai/gpt-5.5`）で指定し、プロバイダーごとに認証して既定モデルを設定する。**50 以上**のプロバイダーに対応する。

## 2 系統のプロバイダー

| 系統 | 設定 | 例 |
|---|---|---|
| **組み込み（pi-ai カタログ）** | コア同梱、認証だけで使える | [[providers/anthropic]]・[[providers/openai]]・[[providers/google]]・OpenRouter・Groq・Mistral・xAI 等 |
| **`models.providers` 経由（カスタム/ベース URL）** | `baseUrl`＋`api` を宣言 | [[providers/ollama]]・LM Studio・[[providers/litellm]]・MiniMax・Volcano・BytePlus 等 |

多くは [[components/plugin-system]] の Plugin として実装される（同梱プロバイダー Plugin）。OpenAI 互換の自前エンドポイントは [[concepts/local-models]] として登録する。

## なぜ重要か

「特定の AI 企業に縛られない」ことが OpenClaw の自由度の源。同じ `provider/model` 抽象の下で、Anthropic からローカル Ollama、統合 Gateway（[[providers/litellm]]）まで差し替えられ、コスト/プライバシー/性能で選べる。プロバイダーは [[concepts/agent-runtimes]] の「provider 層」（モデルを供給する）にあたり、ターンを実行する runtime 層（pi/codex/ACP）とは別軸。

## 認証とフォールバック

- **認証**：API キー（環境変数 or `models.providers.<id>.apiKey`、SecretRef 可）/ サブスク OAuth（[[concepts/oauth]]、例 OpenAI Codex・Claude CLI）/ AWS 認証情報チェーン（[[providers/bedrock]]）。複数アカウントの認証プロファイルをローテーション。
- **フォールバック**：認証プロファイルのローテーション → モデルフォールバックチェーンの 2 段階（[[concepts/model-failover]]）。
- **選択**：`agents.defaults.model.primary`＋`fallbacks`、`/model`（[[concepts/slash-commands]]）、`openclaw models list|status`。

## 既存 wiki とのつながり

プロバイダーは [[concepts/agent-runtimes]] の層分け（provider/model/runtime/channel）の provider/model を埋める。`models.mode: "merge"` でホスト型とローカルを併存（[[concepts/local-models]]）、障害時の挙動は [[concepts/model-failover]]、サブスク認証は [[concepts/oauth]]。新プロバイダーは [[sources/plugins/sdk-provider-plugins]] で自作できる。

## 代表ソース

- [[sources/concepts/model-providers]] — 全プロバイダーの設定リファレンス（組み込み＋カスタム）
- [[sources/concepts/models]] — モデル選択ルールと `openclaw models` CLI
- [[sources/providers/providers]] / [[sources/providers/models]] — プロバイダーディレクトリ/クイックスタート

## 関連ページ

- [[concepts/agent-runtimes]] / [[concepts/model-failover]] / [[concepts/oauth]] / [[concepts/local-models]]
- [[components/gateway]] / [[components/plugin-system]]
- [[providers/anthropic]] / [[providers/openai]] / [[providers/google]] / [[providers/ollama]] / [[providers/bedrock]] / [[providers/litellm]]
