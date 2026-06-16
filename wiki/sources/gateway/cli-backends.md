---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/cli-backends
source_path: raw/docs/gateway/cli-backends.md
doc_section: gateway
title: "CLI バックエンド"
ingested: 2026-06-14
tags: [cli-backends, fallback, claude-cli, codex-cli, jsonl, mcp, text-only]
related:
  - "[[concepts/local-models]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/retry]]"
---

# CLI バックエンド（解説）

> 原典: `raw/docs/gateway/cli-backends.md` ・ https://docs.openclaw.ai/ja-JP/gateway/cli-backends

## 一言まとめ

API プロバイダーが落ちている/レート制限/不安定なときの**テキスト専用フォールバック**として、ローカルの AI CLI（`codex-cli`・`claude-cli`・`google-gemini-cli` 等）を実行する仕組み。意図的に保守的で「外部 API に依存せず常に動く返信」が目的のセーフティネット。

## 位置づけ

[[concepts/local-models]] と並ぶ「ローカル実行」の一形態だが、本質は [[concepts/agent-runtimes]] の**バックエンド/ハーネス層**の一種で、[[concepts/retry]] のモデルフェイルオーバー末端に置く。**ACP ではない**（完全なハーネスランタイムが要るなら [[concepts/acp]] / [[sources/tools/acp-agents]]）。

## 仕組み・ふるまい

1. プロバイダープレフィックス（`codex-cli/...`）でバックエンド選択 → 2. 同じ OpenClaw プロンプト＋ワークスペース文脈でシステムプロンプト構築 → 3. 対応すればセッション ID 付きで CLI 実行 → 4. JSON/JSONL/テキストを解析して最終テキスト抽出 → 5. バックエンドごとにセッション ID を永続化して後続ターン再利用。

- **同梱 `claude-cli`**：再びサポート（Anthropic から `claude -p` 使用を認可済みと通知）。OpenClaw セッションごとに Claude stdio プロセスを warm 維持（`liveSession: "claude-stdio"`・`output:"jsonl"`・`input:"stdin"`）、後続を stream-json stdin で送信。Gateway 再起動/アイドル終了時は保存セッション ID から `--resume`（トランスクリプト検証つき）。1 ターン上限 8MiB/20,000 行（最大 64MiB/100,000 行までクランプ）。OpenClaw の `/think` を Claude の `--effort` に、exec ポリシーが YOLO なら `--permission-mode bypassPermissions` にマップ。
- **同梱 `codex-cli`**：設定なしで使える（`openclaw agent --model codex-cli/gpt-5.5`）。`model_instructions_file` 経由でシステムプロンプトを一時ファイル渡し。
- **画像**：`imageArg` 設定で base64 を一時ファイル化して渡す（なければプロンプトにパス注入）。
- **バンドル MCP オーバーレイ**：`bundleMcp: true` で loopback HTTP MCP サーバーを起動し Gateway ツールを CLI に公開（セッショントークン `OPENCLAW_MCP_TOKEN` で認証、セッション/アカウント/チャネルにスコープ）。

## 設定・使い方の要点

- すべて `agents.defaults.cliBackends.<provider-id>` 配下。`command`（最小 PATH 環境では絶対パス）・`args`・`output`（json/jsonl/text）・`input`（arg/stdin）・`modelArg`/`modelAliases`・`sessionArg`/`sessionMode`（always/existing/none）・`systemPromptArg`・`serialize`。デフォルトは所有 Plugin が `api.registerCliBackend(...)` で登録（ユーザー設定が上書き）。
- フォールバック用途：`model.fallbacks: ["codex-cli/gpt-5.5"]`。許可リスト `agents.defaults.models` 使用時はそこにも追加。
- `claude-cli` 前提：同一ホストで Claude Code がログイン済み（`claude auth login` → `openclaw models auth login --provider anthropic --method cli`）。

## 注意点・落とし穴

- ⚠️ **OpenClaw ツール呼び出しは直接注入されない**（`bundleMcp: true` のときのみ Gateway ツールを認識）。ストリーミング有無はバックエンド固有。Codex CLI セッションはテキストで再開（JSONL より構造化度が低い）。
- 認証 ID（プロファイル ID・静的キー・OAuth アカウント ID）が変わると保存 CLI セッションの再利用を破棄（トークンローテーション程度では切らない）。

## 用語と略称

- **CLI バックエンド** = ローカル AI CLI をテキスト専用フォールバックとして実行する仕組み
- **JSONL** = JSON Lines（1 行 1 JSON のストリーム形式）
- **MCP** = Model Context Protocol（ツールを外部に公開する標準）
- **ACP** = Agent Client Protocol（完全なハーネスランタイム制御プロトコル。CLI バックエンドとは別物）
- **stream-json / warm stdio** = 常駐させた stdio プロセスへ逐次 JSON を流す方式

## 関連ページ

- [[concepts/local-models]] / [[concepts/agent-runtimes]]
- [[concepts/retry]] / [[concepts/session]] / [[components/gateway]]
