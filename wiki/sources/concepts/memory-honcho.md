---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/memory-honcho
source_path: raw/docs/concepts/memory-honcho.md
doc_section: concepts
title: "Honcho メモリ"
ingested: 2026-06-14
tags: [memory, backend, honcho, cross-session, user-modeling, plugin]
related:
  - "[[concepts/memory]]"
  - "[[concepts/context-engine]]"
---

# Honcho メモリ（解説）

> 原典: `raw/docs/concepts/memory-honcho.md` ・ https://docs.openclaw.ai/ja-JP/concepts/memory-honcho

## 一言まとめ

Honcho は OpenClaw に **AI ネイティブなメモリ**を足す Plugin。会話を専用サービスに永続化し、時間をかけて**ユーザーとエージェントのモデル（プロファイル）を自動構築**することで、ワークスペースの Markdown を超えたクロスセッションコンテキストを提供する。

## 位置づけ

[[concepts/memory]] のバックエンドの 1 つだが、組み込み/QMD が「ワークスペースの Markdown ファイル」を検索するのに対し、Honcho は**専用サービス**に会話を持ち、自動でユーザーモデリングする点が違う。並用も可能（QMD があれば Honcho のクロスセッションと並行してローカル Markdown も検索）。コンテキスト注入は [[concepts/context-engine]] の `before_prompt_build` フェーズで行われる。

## 仕組み・ふるまい

- **クロスセッションメモリ**：会話は各ターン後に永続化され、セッションリセット・[[concepts/compaction]]・チャネル切り替えをまたいでコンテキストを維持。
- **ユーザーモデリング**：ユーザーのプロファイル（好み・事実・コミュニケーションスタイル）とエージェントのプロファイル（性格・学習された振る舞い）を維持。
- **セマンティック検索**：現在セッションだけでなく過去会話の観察内容を検索。
- **マルチエージェント対応**：親は生成したサブエージェントを自動追跡し、親が子の observer として追加される。
- 注入タイミング：会話中、Honcho ツールが `before_prompt_build` でサービスに問い合わせ、モデルがプロンプトを見る前に関連コンテキストを注入する。
- ツール：取得系（`honcho_context`/`honcho_search_conclusions`/`honcho_search_messages`/`honcho_session`、LLM 呼び出し無しで高速）と Q&A 系（`honcho_ask`、`depth='quick'|'thorough'`）。

## 設定・使い方の要点

- 導入：`openclaw plugins install @honcho-ai/openclaw-honcho` → `openclaw honcho setup`（API 認証情報を尋ね config を書き、既存ワークスペースメモリを**非破壊**で移行）→ `openclaw gateway --force`。
- 設定は `plugins.entries["openclaw-honcho"].config`（`apiKey`/`workspaceId`/`baseUrl`）。**完全ローカル（セルフホスト）**なら `baseUrl` をローカルサーバーに向け `apiKey` 省略（外部依存なし）。
- CLI：`openclaw honcho setup|status|ask <q>|search <query>`。

## 注意点・落とし穴

- 組み込み/QMD との比較：ストレージは Markdown ファイル vs 専用サービス、クロスセッションは手動 vs 自動、ユーザーモデリングは手動（`MEMORY.md` に書く）vs 自動プロファイル、依存は無し/QMD バイナリ vs Plugin インストール。
- マネージド API（`api.honcho.dev`）利用時は会話が外部サービスへ送られる点に留意（セルフホストなら外部依存なし）。

## 用語と略称

- **クロスセッションメモリ** = セッションをまたいで維持される記憶
- **ユーザーモデリング** = ユーザー/エージェントのプロファイルを継続的に構築すること
- **observer** = 他セッションの会話を観測する役割（マルチエージェントの親）
- **セルフホスト** = 自前サーバーで動かす構成

## 関連ページ

- [[concepts/memory]] / [[concepts/context-engine]]
- [[sources/concepts/memory-builtin]] / [[sources/concepts/memory-qmd]]
