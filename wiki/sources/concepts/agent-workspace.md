---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/agent-workspace
source_path: raw/docs/concepts/agent-workspace.md
doc_section: concepts
title: "エージェントのワークスペース"
ingested: 2026-06-14
tags: [workspace, bootstrap, memory, git, sandbox]
related:
  - "[[concepts/agent-workspace]]"
  - "[[concepts/agent]]"
  - "[[concepts/soul]]"
---

# エージェントのワークスペース（解説）

> 原典: `raw/docs/concepts/agent-workspace.md` ・ https://docs.openclaw.ai/ja-JP/concepts/agent-workspace

## 一言まとめ

ワークスペースは「エージェントのホーム」――ファイルツールとワークスペースコンテキストが使う**唯一の作業ディレクトリ（cwd）**であり、**非公開メモリ**として扱うべき場所。ここに置くべき標準ファイル群と、その場所・バックアップ・新マシンへの移行方法を説明したページ。

## 位置づけ

[[concepts/agent]]（単一エージェントプロセスの契約）が「ワークスペースが要る」と述べた、その**ワークスペースそのものの詳細**。中の `SOUL.md` の書き方は [[concepts/soul]]、これらのファイルが実際にどうプロンプトへ注入されるかは [[concepts/system-prompt]] に続く。設定・認証・セッションを置く `~/.openclaw/` とは**別物**である点が重要。

## 仕組み・ふるまい

### 場所

- 既定 `~/.openclaw/workspace`（`OPENCLAW_PROFILE` が `default` 以外なら `~/.openclaw/workspace-<profile>`）。`~/.openclaw/openclaw.json` の `agents.defaults.workspace` で上書き。
- `openclaw onboard` / `configure` / `setup` がワークスペースを作成し、欠けたブートストラップファイルを初期配置する（`skipBootstrap: true` で無効化）。

### ワークスペースファイルマップ（標準ファイル）

- `AGENTS.md`（操作手順＋メモリの使い方）、`SOUL.md`（ペルソナ・トーン・境界 → [[concepts/soul]]）、`USER.md`（ユーザー像）、`IDENTITY.md`（名前/雰囲気/絵文字）、`TOOLS.md`（ローカルツール規約）。
- `HEARTBEAT.md`（Heartbeat 用チェックリスト）、`BOOT.md`（Gateway 再起動時に自動実行する起動チェックリスト、内部フックが有効な場合）、`BOOTSTRAP.md`（初回の一度きり儀式、新ワークスペースのみ）。
- `memory/YYYY-MM-DD.md`（日次メモリログ）、`MEMORY.md`（整理済み長期メモリ、任意。メインの非公開セッションのみ読み込む）。
- `skills/`（ワークスペース Skills、最優先）、`canvas/`（Node 表示用 Canvas UI）。

### ワークスペースに含まれないもの（`~/.openclaw/` 配下）

設定 `openclaw.json`、モデル認証 `auth-profiles.json`（OAuth＋API キー → [[concepts/oauth]]）、`codex-home/`、`credentials/`、`sessions/`、管理対象 `skills/`。これらはワークスペースのリポジトリにコミットしない。

## 設定・使い方の要点

- **Git バックアップ（推奨・private）**：ワークスペースを private git リポジトリに置き、`AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md memory/` をコミット。新マシンへはクローン → `agents.defaults.workspace` を指す → `openclaw setup --workspace <path>` で欠けたファイルを補完。
- ブートストラップ注入の上限：ファイルごと `bootstrapMaxChars`（既定 12000）、合計 `bootstrapTotalMaxChars`（既定 60000）。

## 注意点・落とし穴

- ⚠️ ワークスペースは**既定の cwd であって強固なサンドボックスではない**。ツールは相対パスをワークスペース基準で解決するが、サンドボックス（[[concepts/sandboxing]]）が無効なら絶対パスでホストの他所にも到達できる。分離が要るなら `agents.defaults.sandbox` を使う。
- **アクティブなワークスペースは 1 つだけに**。古い `~/openclaw` などを残すと認証・状態のずれで混乱する（`openclaw doctor` が警告）。
- **秘密情報をコミットしない**（private リポジトリでも）。API キー・OAuth トークン・パスワードはプレースホルダー化し、実体は `~/.openclaw/` やパスワードマネージャーへ。

## 用語と略称

- **cwd** = current working directory（カレント作業ディレクトリ）
- **ブートストラップファイル** = 各セッション開始時にシステムプロンプトへ注入される、人格・手順・ユーザー像の markdown 群
- **Heartbeat** = 定期実行でエージェントに自己点検させる仕組み

## 関連ページ

- [[concepts/agent-workspace]] — 対応する概念ページ
- [[concepts/agent]] / [[concepts/soul]] / [[concepts/system-prompt]]
- [[concepts/session]] / [[concepts/memory]] / [[concepts/sandboxing]]
