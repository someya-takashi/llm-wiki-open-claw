---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/agent-runtimes
source_path: raw/docs/concepts/agent-runtimes.md
doc_section: concepts
title: "エージェントランタイム（実行バックエンドの層）"
ingested: 2026-06-14
tags: [agent-runtime, harness, codex, pi, claude-cli, acp]
related:
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/agent]]"
  - "[[concepts/model-providers]]"
---

# エージェントランタイム（実行バックエンドの層）（解説）

> 原典: `raw/docs/concepts/agent-runtimes.md` ・ https://docs.openclaw.ai/ja-JP/concepts/agent-runtimes
>
> ⚠️ 紛らわしいが、本ページ（concepts/agent-runtimes）は **ターンを実行するバックエンドの「層」**。1 プロセスの構成・セッション契約は別ドキュメント [[concepts/agent]] を参照。

## 一言まとめ

**エージェントランタイム** とは「準備済みのモデルループを 1 つ所有し、プロンプトを受けてモデル出力を駆動し、ネイティブツール呼び出しを処理して完了ターンを OpenClaw に返す」コンポーネント。`pi`・`codex`・`claude-cli` といった**実行バックエンド**を指し、プロバイダーやモデルとは別レイヤーであることを整理したページ。

## 位置づけ

[[concepts/agent-loop]] が「1 ターンをどう配線するか」だとすると、本ページは「その 1 ターンを実際に回す**エンジンをどれにするか**」を扱う。混同しやすい 4 つのレイヤーを分離する：

| レイヤー | 例 | 意味 |
| --- | --- | --- |
| プロバイダー | `openai`, `anthropic` | 認証・モデル検出・モデル参照の名付け（→ [[concepts/model-providers]]） |
| モデル | `gpt-5.5`, `claude-opus-4-6` | そのターンに選ぶモデル |
| エージェントランタイム | `pi`, `codex`, `claude-cli` | 準備済みターンを実行する低レベルのループ／バックエンド |
| チャネル | Telegram, Discord, Slack | メッセージの出入り口（→ channels/） |

コード上の **ハーネス（harness）** は「エージェントランタイムを提供する実装」。例：バンドルされた Codex ハーネスは `codex` ランタイムを実装する。

## 仕組み・ふるまい

### 2 つのランタイムファミリー

- **埋め込みハーネス**：OpenClaw の準備済みエージェントループ内で走る。組み込みの `pi` に加え、`codex` などの登録済み Plugin ハーネス。
- **CLI バックエンド**：モデル参照を正規に保ったままローカル CLI プロセスを実行する。例：`anthropic/claude-opus-4-7` にモデルスコープの `agentRuntime.id: "claude-cli"` を付けると「Anthropic モデルを選び、Claude CLI 経由で実行」を意味する。`claude-cli` は埋め込みハーネス ID ではない。

### Codex の複数サーフェス（混乱しやすい）

「Codex」という名前は意図的に独立した複数の面で共有される：ネイティブ Codex app-server ランタイム（`openai/*` モデル参照）、Codex OAuth 認証プロファイル（`openai-codex` 認証プロバイダー）、Codex **ACP（Agent Client Protocol）** アダプター（`runtime:"acp"`, `agentId:"codex"`）、ネイティブの `/codex ...` チャット制御コマンド、エージェント以外用の OpenAI Platform API ルート。`codex` Plugin を有効化するとネイティブ app-server 機能が使える。

### ランタイムの所有権

ランタイムごとに「ループのどこを所有するか」が違う。PI 埋め込みは OpenClaw がモデルループ・正規スレッド状態・ツール・コンテキストエンジン・Compaction を所有。Codex app-server は Codex 側がモデルループと（OpenClaw がミラーする）スレッド状態・ネイティブツール・Compaction を所有し、OpenClaw はコンテキストを組み立てて投入しチャネル配信を担う。**この所有権の分割が主な設計ルール**で、ネイティブランタイムが状態を所有する箇所では OpenClaw は書き換えずミラー＆投影する。

### ランタイムの選択

プロバイダーとモデルを解決した後に埋め込みランタイムを選ぶ：①モデルスコープのランタイムポリシー（`agentRuntime`）が最優先 → ②プロバイダースコープのポリシー → ③`auto` で登録 Plugin ランタイムが対応を要求 → ④誰も要求しなければ互換ランタイムとして **PI** にフォールバック。明示的なプロバイダー/モデルランタイムは**フェイルクローズ**（暗黙に PI へ戻らない）。`auto` は保守的だが OpenAI エージェントモデルは例外で、未設定/`auto` はどちらも Codex ハーネスに解決される。

## 設定・使い方の要点

- **セッション全体・エージェント全体のランタイム固定は無視される**（`OPENCLAW_AGENT_RUNTIME`、`agents.defaults.agentRuntime` 等を含む）。固定はモデルスコープ/プロバイダースコープのポリシーで行う。
- レガシー設定の整理は `openclaw doctor --fix`：古いエージェント全体のランタイム固定を削除し、`openai-codex/*` モデル参照を `openai/*`＋モデルスコープ `agentRuntime.id: "codex"` に書き換える。
- 一般的な ChatGPT/Codex サブスク構成：認証は Codex OAuth、モデル参照は `openai/*` のまま、ランタイムは `codex`。Claude Code・Gemini CLI・OpenCode・Cursor 等の外部ハーネスは ACP/acpx を使う。

## 注意点・落とし穴

- **ランタイム ≠ プロバイダー ≠ モデル ≠ チャネル**。ステータス出力の `Execution`/`Runtime` ラベルは診断情報であり、プロバイダー名と読み違えない。
- `openai/*` を選ぶことは（OpenAI API サーフェスを使っていない限り）現状「Codex 経由で実行する」を意味する。「API 課金を使う」という意味ではない。
- 想定外のランタイムが出るときは、まず選択中プロバイダー/モデルのランタイムポリシーを確認する（レガシーのセッション固定はもうルーティングを決めない）。

## 用語と略称

- **ハーネス（harness）** = エージェントランタイムを提供する実装
- **ACP** = Agent Client Protocol（外部エージェントを制御する規格。`acpx` はその実装系）
- **OAuth** = 認証情報を安全に委譲する認可規格
- **フェイルクローズ** = 条件を満たせないとき安全側（実行しない/エラー）に倒す挙動
- **app-server** = Codex がターンを実行するサーバープロセス

## 関連ページ

- [[concepts/agent-runtimes]] — 本ドキュメントに対応する概念ページ
- [[concepts/agent]] — 1 プロセスの契約（混同注意）
- [[concepts/agent-loop]] — 実行サイクル本体
- [[concepts/model-providers]] / [[providers/openai]] / [[providers/anthropic]]
