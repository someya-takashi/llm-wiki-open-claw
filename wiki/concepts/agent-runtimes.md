---
type: concept
aliases: [Agent runtimes, harness, ハーネス, runtime]
tags: [agent-runtime, harness, codex, pi, claude-cli, acp]
related:
  - "[[concepts/agent]]"
  - "[[concepts/agent-loop]]"
  - "[[concepts/model-providers]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/agent-runtimes]]"
updated: 2026-06-14
---

# エージェントランタイム（実行バックエンドの層）

> ⚠️ **混同注意**：本ページ（`agent-runtimes`）は **ターンを実行するバックエンドの層**。1 プロセスの構成・セッション契約は [[concepts/agent]] を見ること。

**エージェントランタイム**とは「準備済みのモデルループを 1 つ所有し、プロンプトを受けてモデル出力を駆動し、完了ターンを返す」交換可能なバックエンド――`pi`（組み込み）、`codex`、`claude-cli` など。コード上は **ハーネス（harness, ランタイムを提供する実装）** とも呼ぶ。

## なぜ分けて考えるのか

設定上、ランタイムはプロバイダー・モデルの近くに現れるため混同されやすいが、これらは**別レイヤー**である：

| レイヤー | 例 | 役割 |
| --- | --- | --- |
| プロバイダー | `openai`, `anthropic` | 認証・モデル検出・命名（→ [[concepts/model-providers]]） |
| モデル | `gpt-5.5`, `claude-opus-4-6` | そのターンのモデル |
| **エージェントランタイム** | `pi`, `codex`, `claude-cli` | ターンを**実行するループ** |
| チャネル | Telegram, Slack | メッセージの出入り口 |

この分離が効くのは、「Anthropic のモデルを選びつつ実行は Claude CLI に任せる」「OpenAI モデルを Codex app-server で回す」といった**モデルと実行系の自由な組み合わせ**を可能にするから。誰が何を所有するか（モデルループ・正規スレッド状態・ツール・Compaction）が **OpenClaw 所有か／ネイティブランタイム所有か**で、Plugin フックの効き方やミラー要否が決まる――これが主要な設計ルール。

## 押さえる点

- **固定はモデル/プロバイダースコープで**。セッション全体・エージェント全体のランタイム固定は無視され、`openclaw doctor --fix` がレガシー設定を正規化する。
- 選択順：モデルスコープ → プロバイダースコープ → `auto`（Plugin が要求）→ 互換の **PI** フォールバック。明示指定は**フェイルクローズ**。
- 「Codex」は app-server ランタイム／OAuth 認証／ACP アダプター／`/codex` コマンド等、独立した複数サーフェスの総称。詳細・判断ツリーは [[sources/concepts/agent-runtimes]]。Codex ランタイムは同梱 `codex` Plugin（[[components/plugin-system]]）として実装され、セットアップは [[sources/plugins/codex-harness]]、デスクトップ制御は [[sources/plugins/codex-computer-use]]、ネイティブ Codex プラグインは [[sources/plugins/codex-native-plugins]]。

プロバイダーや CLI バックエンドは [[components/plugin-system]] の Plugin として実装する：作り方は [[sources/plugins/sdk-provider-plugins]]（モデルプロバイダー）・[[sources/plugins/cli-backend-plugins]]（テキスト CLI フォールバック）、コアの共有ドメイン追加は [[sources/plugins/adding-capabilities]]。外部のコーディングハーネス（Claude Code 等）を丸ごと走らせる完全なランタイムは **[[concepts/acp]]**（Agent Client Protocol）。

## 関連

- [[concepts/agent]] — 1 プロセスの契約（混同注意）
- [[concepts/agent-loop]] — 実行サイクル本体 / [[concepts/acp]] — 外部ハーネスランタイム
- [[concepts/model-providers]] / [[concepts/model-failover]] / [[providers/openai]] / [[providers/anthropic]]
- [[components/plugin-system]] / [[sources/plugins/codex-harness]]
