---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/agent
source_path: raw/docs/concepts/agent.md
doc_section: concepts
title: "エージェントランタイム（単一プロセスの契約）"
ingested: 2026-06-14
tags: [agent, workspace, bootstrap, session, skills]
related:
  - "[[concepts/agent]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/agent-loop]]"
---

# エージェントランタイム（単一プロセスの契約）（解説）

> 原典: `raw/docs/concepts/agent.md` ・ https://docs.openclaw.ai/ja-JP/concepts/agent
>
> ⚠️ 紛らわしいが、本ページ（concepts/agent）は **1 つのエージェントプロセスの契約**。ターンを実行するバックエンドの「層」（pi/codex/claude-cli）は別ドキュメント [[concepts/agent-runtimes]] を参照。

## 一言まとめ

OpenClaw が [[components/gateway]] ごとに走らせる **単一の埋め込みエージェントランタイム** が、何をワークスペースとして使い・どのファイルをシステムプロンプトに差し込み・セッションをどう保存するか、という「1 プロセス分の契約」を定義したページ。

## 位置づけ

[[components/gateway]] は 1 プロセスのエージェントを抱える。そのプロセスが「どこで動き（ワークスペース）」「どんな人格・手順を読み（ブートストラップファイル）」「会話をどこに残すか（セッション）」を決めるのが本ページ。実際にターンを 1 回まわす配線は [[concepts/agent-loop]]、その実行を担うバックエンドの選択は [[concepts/agent-runtimes]] にある。

## 仕組み・ふるまい

### ワークスペース（必須）

- 単一のワークスペースディレクトリ（`agents.defaults.workspace`）が、ツールとコンテキストにとってエージェントの**唯一の作業ディレクトリ（cwd, current working directory）**になる。
- サンドボックス（`agents.defaults.sandbox`, ツール実行を隔離する仕組み）が有効なら、main 以外のセッションは `sandbox.workspaceRoot` 配下のセッションごとワークスペースで上書きできる。
- 未設定時は `openclaw setup` で `~/.openclaw/openclaw.json` とワークスペースファイルを初期化するのが推奨。

### ブートストラップファイル（システムプロンプトへ挿入）

ワークスペース内の以下のユーザー編集可能ファイルが、新セッション最初のターンでシステムプロンプトの Project Context に挿入される：

- `AGENTS.md`（操作手順＋「メモリ」）、`SOUL.md`（ペルソナ・境界・トーン）、`TOOLS.md`（ツールの使い方メモ）、`IDENTITY.md`（名前/雰囲気/絵文字）、`USER.md`（ユーザープロファイルと呼びかけ方）。
- `BOOTSTRAP.md`（初回実行の一度きりの儀式。完了後に削除）は、**完全に新しいワークスペース**のときだけ作成され、完了後に削除すれば再作成されない。
- 空ファイルはスキップ、大きいファイルはプロンプトを軽くするため短縮・切り詰めされる。`agents.defaults.skipBootstrap: true` で挿入を完全停止できる。

### 組み込みツールと Skills

- コアツール（read/exec/edit/write と関連システムツール）は常にツールポリシー対象として利用可能。`apply_patch` は任意（`tools.exec.applyPatch`）。`TOOLS.md` は「どのツールが存在するか」ではなく「どう使ってほしいか」のガイダンス。
- **Skills** は優先度順に複数の場所から読み込まれる：`<workspace>/skills` → `<workspace>/.agents/skills` → `~/.agents/skills` → `~/.openclaw/skills` → バンドル済み → `skills.load.extraDirs`。

### ランタイム境界とセッション

- 埋め込みランタイムは **Pi エージェントコア**（モデル・ツール・プロンプトパイプライン）の上に建つ。セッション管理・検出・ツール配線・チャネル配信は、その上にある OpenClaw 所有のレイヤー。
- セッショントランスクリプトは **JSONL**（1 行 1 JSON のログ形式）で `~/.openclaw/agents/<agentId>/sessions/<SessionId>.jsonl` に保存。セッション ID は OpenClaw が選ぶ安定値。

## 設定・使い方の要点

- **ストリーミング中のステアリング**（実行中の割り込み投入）はキューモードに依存：`steer` は現在のアシスタントターンがツール実行を終えた後・次の LLM 呼び出し前にまとめて配信、`followup`/`collect` はターン終了まで保持して新ターンを開始する（[[concepts/agent-loop]] とキュー側で詳述）。
- **ブロックストリーミング**は既定オフ（`blockStreamingDefault: "off"`）。`blockStreamingBreak`（text_end / message_end）・`blockStreamingChunk`（既定 800–1200 文字）・`blockStreamingCoalesce` で整形。Telegram 以外は `*.blockStreaming: true` の明示が要る。
- **モデル参照**は最初の `/` で分割：`provider/model`。プロバイダー省略時はエイリアス→一意一致プロバイダー→既定プロバイダーの順に解決。OpenRouter 形式（モデル ID に `/` を含む）はプロバイダープレフィックスを付ける（例 `openrouter/moonshotai/kimi-k2`）。

## 注意点・落とし穴

- ワークスペースは**唯一の cwd**。複数ディレクトリをまたぐ前提で設計しない。
- 他ツール由来のレガシーなセッションフォルダは読まれない。
- ブートストラップ挿入は新ワークスペース判定に依存するため、既存ワークスペースに後から `BOOTSTRAP.md` を置いても期待どおり動かないことがある。

## 用語と略称

- **cwd** = current working directory（カレント作業ディレクトリ）
- **JSONL** = JSON Lines（1 行 1 JSON のログ形式）
- **Pi エージェントコア** = OpenClaw が組み込むエージェント実行基盤（モデル/ツール/プロンプト）
- **ステアリング** = 実行中のエージェントに割り込みでメッセージを差し込む操作

## 関連ページ

- [[concepts/agent]] — 本ドキュメントに対応する概念ページ
- [[concepts/agent-runtimes]] — 実行バックエンドの層（混同注意）
- [[concepts/agent-loop]] — 1 ターンの実行配線
- [[concepts/agent-workspace]] / [[concepts/session]] / [[concepts/multi-agent]]
