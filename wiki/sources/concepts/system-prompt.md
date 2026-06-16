---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/system-prompt
source_path: raw/docs/concepts/system-prompt.md
doc_section: concepts
title: "システムプロンプト"
ingested: 2026-06-14
tags: [system-prompt, prompt, bootstrap, cache, skills, prompt-mode]
related:
  - "[[concepts/system-prompt]]"
  - "[[concepts/context]]"
  - "[[concepts/agent-workspace]]"
---

# システムプロンプト（解説）

> 原典: `raw/docs/concepts/system-prompt.md` ・ https://docs.openclaw.ai/ja-JP/concepts/system-prompt

## 一言まとめ

OpenClaw はエージェント実行ごとに**自前のシステムプロンプトを組み立てて注入する**（pi-coding-agent の既定プロンプトは使わない）。そのプロンプトが何のセクションで構成され、どのワークスペースファイルが注入され、どこにプロンプトキャッシュ境界が引かれるかを説明したページ。

## 位置づけ

[[concepts/context]]（モデルに送る全体）の中の最大の固定要素がシステムプロンプト。中身の人格・手順は [[concepts/agent-workspace]] のブートストラップファイル由来で、実行サイクル [[concepts/agent-loop]] の「プロンプト組み立て」段で確定する。実行バックエンド（[[concepts/agent-runtimes]]）が Codex の場合はランタイム側も独自にワークスペースコンテキストを足す。

## 仕組み・ふるまい

### 3 レイヤーの組み立て

1. `buildAgentSystemPrompt`：明示入力からプロンプトをレンダリングする純粋なレンダラー（グローバル設定は直接読まない）。
2. `resolveAgentSystemPromptConfig`：所有者表示・TTS ヒント・モデルエイリアス・メモリ引用モード・サブエージェント委任モードなど、設定由来の調整を解決。
3. ランタイムアダプター（埋め込み/CLI/プレビュー/Compaction）：ツール・サンドボックス状態・チャネル機能・コンテキストファイル・プロバイダーのプロンプト寄与などのライブ情報を集めてファサードを呼ぶ。

これで「エクスポート/デバッグ表示」と「ライブ実行」を一致させつつ、巨大な単一ビルダー化を避ける。

### 固定セクション（抜粋）

ツール、実行バイアス（やり切りガイダンス）、安全性（助言的ガードレール）、Skills、OpenClaw 制御／自己更新（`gateway` ツールや `config.*` の使い方）、ワークスペース、ドキュメント、注入済みワークスペースファイル、サンドボックス、現在の日付と時刻（タイムゾーンのみ）、アシスタント出力指示、Heartbeats、ランタイム、推論。

### プロンプトキャッシュ境界

**Project Context を含む大きな安定コンテンツは内部キャッシュ境界の上**に置き、メッセージング・音声・グループチャット・リアクション・Heartbeats・ランタイムなど**揮発的なチャネル/セッションセクションは境界の下**に追加する。これでプレフィックスキャッシュを持つローカルバックエンドが、チャネルターンをまたいで安定プレフィックスを再利用できる。

### ワークスペースブートストラップ注入

`AGENTS.md SOUL.md TOOLS.md IDENTITY.md USER.md HEARTBEAT.md`（＋新規時のみ `BOOTSTRAP.md`、存在すれば `MEMORY.md`）が **Project Context の下に毎ターン注入**される。`HEARTBEAT.md` は Heartbeat 無効時は省略。Codex ハーネスでは `AGENTS.md` は Codex 側が読むため二重化せず、残りは Codex 設定指示として転送。`memory/*.md` の日次ファイルは通常ターンでは注入されず `memory_search`/`memory_get` でオンデマンド取得（裸の `/new` `/reset` 初回は例外）。

## 設定・使い方の要点

- **プロンプトモード**（ランタイムが各実行に設定、ユーザー設定ではない）：`full`（既定・全部）、`minimal`（サブエージェント用に多くを省略）、`none`（ベース ID 行のみ）。
- 注入上限：`bootstrapMaxChars`（既定 12000）/`bootstrapTotalMaxChars`（既定 60000）。切り詰め警告は `bootstrapPromptTruncationWarning`（off/once/always、既定 once）。
- 内訳確認：`/context list` `/context detail`（→ [[concepts/context]]）。
- ドキュメントセクションはローカル `docs/` を指し、無ければ docs.openclaw.ai にフォールバック。設定はまず `gateway` ツールの `config.schema.lookup` を引くようモデルに指示する。

## 注意点・落とし穴

- **安全性ガードレールは助言であって強制ではない**。強制にはツールポリシー・exec 承認・サンドボックス・チャネル許可リストを使う（オペレーターは設計上これらを無効化できる）。
- `MEMORY.md` が繰り返し切り詰められるなら、短い要約に蒸留し詳細は `memory/*.md` へ移す。
- プロンプトスナップショット（`test/fixtures/.../prompt-snapshots/`）でドリフトを CI 検証する。

## 用語と略称

- **プロンプトキャッシュ境界** = この上の安定プレフィックスをキャッシュ再利用するための区切り
- **promptMode** = 実行ごとのプロンプト詳細度（full/minimal/none）
- **Project Context** = ブートストラップファイルが注入されるシステムプロンプト内の領域
- **TTS** = Text-to-Speech（音声合成）

## 関連ページ

- [[concepts/system-prompt]] — 対応する概念ページ
- [[concepts/context]] / [[concepts/agent-workspace]] / [[concepts/soul]]
- [[concepts/agent-loop]] / [[concepts/agent-runtimes]]
