---
type: concept
aliases: [Local Models, ローカルモデル, セルフホストモデル]
tags: [local-models, self-hosted, openai-compat, lm-studio, ollama, privacy]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/retry]]"
  - "[[concepts/security]]"
  - "[[concepts/compaction]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/local-models]]"
  - "[[sources/gateway/local-model-services]]"
  - "[[sources/gateway/cli-backends]]"
updated: 2026-06-14
---

# ローカルモデル（Local Models）

ローカルモデルは、OpenClaw を**自前ホストの LLM（大規模言語モデル）で動かす**こと。データを外に出さないプライバシー最強の経路だが、ハードウェア・コンテキスト長・プロンプトインジェクション防御の要求水準が上がる。一貫した指針は「**ホストできる最大/フルサイズを使え、小型・過量子化は避けろ**」。

## 位置づけ

[[concepts/agent-runtimes]] が整理する provider/model/runtime/channel の層のうち、**provider/model 層をローカルサーバーに差し替える**話。ローカルサーバーは OpenAI 互換の `/v1` を話す限り何でもよく、`models.providers.<id>` に登録する（[[concepts/configuration]]）。OpenClaw 側から見れば、ホスト型モデルと同じインターフェースで、`models.mode: "merge"` によってローカル⇄ホストのフォールバック（[[concepts/retry]]）を組める。

## 3 つの取り込み方

| 方式 | 内容 | ソース |
|---|---|---|
| **ローカルサーバーを登録** | LM Studio / Ollama / MLX / vLLM / SGLang などを OpenAI 互換プロバイダーとして登録（推奨は LM Studio ＋ Responses API） | [[sources/gateway/local-models]] |
| **オンデマンド起動** | `localService` で「選んだモデルが必要なときだけ」サーバープロセスを起動・停止 | [[sources/gateway/local-model-services]] |
| **CLI バックドフォールバック** | API 障害時にローカル AI CLI（`codex-cli`/`claude-cli`）をテキスト専用で実行 | [[sources/gateway/cli-backends]] |

## 運用の勘所

- **ハードウェア下限**：快適なエージェントループには最大構成 Mac Studio ×2 以上 or 同等 GPU リグ（~$30k+）が目安。単一 24GB GPU は軽量・高レイテンシ用。
- **互換シム**：`compat.requiresStringContent`・`strictMessageKeys`・`supportsTools: false`、ツール強制が要るサーバーには `params.extra_body.tool_choice: "required"`。
- **切り分け**：`openclaw infer model run --local`／`--gateway` でトランスポートを段階確認 →「軽量モード」`experimental.localModelLean`（重い `browser`/`cron`/`message` を外す）→ `compat.supportsTools: false` → それ以降は上流の容量問題。

## 既存 wiki とのつながり

⚠️ **ローカルモデルはプロバイダー側の安全フィルターを通らない**——これが [[concepts/security]] 上の最大の注意点。過量子化・小型はプロンプトインジェクション耐性が落ちるので、エージェントの権限を狭く保ち、[[concepts/compaction]]（長い会話の要約圧縮）を有効にして影響範囲を抑える。フォールバック構成は [[concepts/retry]] のモデルフェイルオーバーそのもので、CLI バックエンドはその末端（テキスト専用セーフティネット）にあたる。

## 代表ソース

- [[sources/gateway/local-models]] — バックエンド選択・推奨スタック・互換シム・切り分け
- [[sources/gateway/local-model-services]] — オンデマンドなサーバープロセス起動
- [[sources/gateway/cli-backends]] — テキスト専用 CLI フォールバック

## 関連ページ

- [[concepts/agent-runtimes]] / [[concepts/retry]] / [[concepts/configuration]]
- [[concepts/security]] / [[concepts/compaction]] / [[components/gateway]]
