---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools
source_path: raw/docs/tools/tools.md
doc_section: tools
title: "ツール（Tools / Skills / Plugins 概要）"
ingested: 2026-06-14
tags: [tools, skills, plugins, capabilities, overview, tool-policy]
related:
  - "[[concepts/session-tool]]"
  - "[[components/plugin-system]]"
  - "[[concepts/sandboxing]]"
---

# ツール（Tools / Skills / Plugins 概要）（解説）

> 原典: `raw/docs/tools/tools.md` ・ https://docs.openclaw.ai/ja-JP/tools
>
> ℹ️ `tools/` セクションのランディング。エージェントの「できること」を増やす 3 つのサーフェス——**ツール / Skills / Plugins**——を選び分けるためのルーティングページ。

## 一言まとめ

**ツール**＝呼び出せるアクション（型付き関数）、**Skills**＝作業のやり方を教える指示パック（`SKILL.md`）、**Plugins**＝ツール/プロバイダー/チャネル/フック等のランタイム機能を追加する拡張。この 3 つの違いと、どれを使うかの判断を示す。

## 位置づけ

[[concepts/session-tool]]（エージェントが呼ぶツール）の上位概観で、拡張機構は [[components/plugin-system]]、ツールの可否制御は [[concepts/sandboxing]]（サンドボックス/ツールポリシー/昇格）。正規のツールポリシーは [[sources/gateway/config-tools]]。

## 仕組み・ふるまい（3 サーフェスの選び分け）

| サーフェス | 使うとき | 例 |
|---|---|---|
| **ツール（tool）** | エージェントが「行動」する（読む/書く/送る/呼ぶ） | `exec`・`browser`・`web_search`・`message`・`image_generate` |
| **Skill**（[[concepts/skills]]） | 必要なツールはあるが「やり方」を教えたい | `SKILL.md` 指示パック（ワークスペース/共有/Plugin 同梱） |
| **Plugin** | OpenClaw に「新しい機能」を足す（コード/資格情報/ライフサイクル） | チャネル・モデルプロバイダー・音声・メディア生成・フック |

- モデルが見るのは、プロファイル・許可/拒否ポリシー・プロバイダー制限・サンドボックス状態・チャネル権限・Plugin 可用性を通過したツールだけ。
- **組み込みツールカテゴリ**：ランタイム（`exec`/`process`/`code_execution`）・ファイル（`read`/`write`/`edit`/`apply_patch`）・Web（`web_search`/`web_fetch`）・ブラウザ・メッセージング（`message`）・セッション/エージェント（`sessions_*`/`subagents`）・Automation（`cron`）・Gateway/ノード（`gateway`/`nodes`）・メディア（`image_generate`/`video_generate`/`tts`）・Tool Search。

## 設定・使い方の要点

- ツールが見えない時の切り分け：アクティブプロファイルと `tools.allow`/`deny` → プロバイダー制限 → サンドボックス/昇格 → 所有 Plugin が有効か → エージェントごと制限 → Tool Search 経由か。
- Plugin 提供ツールの例：Diffs・LLM Task・Lobster・Tokenjuice・Tool Search・Canvas。

## 注意点・落とし穴

- ツールポリシーは**モデル呼び出しの前**に適用され、削られたツールはそのターンのスキーマに載らない。Codex ハーネス実行では `tools.toolSearch` でなく Codex ネイティブのコードモード/ツール検索を使う。

## 用語と略称

- **ツール（tool）** = エージェントが呼べる型付き関数
- **Skill** = `SKILL.md` の指示パック（やり方を教える）
- **Plugin** = ランタイム機能を足す拡張
- **PI** = OpenClaw の組み込みエージェントハーネス（[[concepts/agent-runtimes]]）
- **Tool Search** = 大規模ツールカタログを検索して呼ぶ実験的サーフェス

## 関連ページ

- [[concepts/session-tool]] / [[concepts/skills]] / [[components/plugin-system]] / [[concepts/sandboxing]]
- [[concepts/slash-commands]] / [[sources/tools/plugin]] / [[sources/gateway/config-tools]] / [[concepts/agent-runtimes]]
