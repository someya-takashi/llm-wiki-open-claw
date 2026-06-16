---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/session-tool
source_path: raw/docs/concepts/session-tool.md
doc_section: concepts
title: "セッションツール"
ingested: 2026-06-14
tags: [session-tool, subagent, orchestration, tools, visibility]
related:
  - "[[concepts/session-tool]]"
  - "[[concepts/session]]"
  - "[[concepts/multi-agent]]"
---

# セッションツール（解説）

> 原典: `raw/docs/concepts/session-tool.md` ・ https://docs.openclaw.ai/ja-JP/concepts/session-tool

## 一言まとめ

エージェントが**セッションをまたいで作業し、状態を調査し、サブエージェント（子エージェント）をオーケストレーションする**ためのツール群（`sessions_list` / `sessions_history` / `sessions_send` / `sessions_spawn` / `sessions_yield` / `subagents` / `session_status`）と、その可視性スコープ・ツールポリシーを説明したページ。

## 位置づけ

[[concepts/session]] が「セッションとは何か」だとすると、本ページは「エージェントがセッションを**操作する道具**」。[[concepts/agent-loop]] 上で動くツールで、サブエージェント生成は [[concepts/multi-agent]] の実行面に当たる。どのツールが使えるかは [[concepts/agent]] のツールポリシー（`tools.profile`）が制御する。

## 仕組み・ふるまい

### 利用可能なツール

| ツール | 動作 |
| --- | --- |
| `sessions_list` | フィルター（kind/label/agent/recency/preview）でセッション一覧 |
| `sessions_history` | 特定セッションのトランスクリプトを読む（既定でツール結果除外、`includeTools: true` で含む） |
| `sessions_send` | 別セッションへメッセージ送信（任意で待機） |
| `sessions_spawn` | バックグラウンド作業用に分離サブエージェントを生成（常に非ブロッキング） |
| `sessions_yield` | 現ターンを終え、後続のサブエージェント結果を次メッセージとして待つ |
| `subagents` | 生成済みサブエージェントの一覧/誘導(steer)/終了(kill) |
| `session_status` | `/status` 相当のカード表示、任意でセッション単位のモデル上書き |

### ツールプロファイルによる付与

`tools.profile: "coding"` は `sessions_spawn`/`sessions_yield`/`subagents` を含む完全なオーケストレーションセット。`tools.profile: "messaging"` はセッション間メッセージング（`sessions_list`/`history`/`send`/`session_status`）のみで生成は含まない。両立は `alsoAllow: ["sessions_spawn", "sessions_yield", "subagents"]`。グループ/プロバイダー/サンドボックス/エージェント単位ポリシーは後段でさらに削れる（`/tools` で有効一覧を確認）。

### サブエージェント生成（`sessions_spawn`）

`runId` と `childSessionKey` を返して即終了。主なオプション：`runtime: "subagent"`（既定）/`"acp"`、子の `model`/`thinking` 上書き、`thread: true`（チャットスレッドにバインド）、`sandbox: "require"`、`context: "fork"`（親トランスクリプトを引き継ぐ）/`"isolated"`（クリーンな子）。既定のリーフサブエージェントにセッションツールは付与されず、`maxSpawnDepth >= 2` の深さ 1 オーケストレーターにのみ再帰オーケストレーション用ツールが付く。完了後は通知ステップが結果をリクエスターのチャンネルへ投稿する。

### セッション間メッセージと安全性

`sessions_send` は `timeoutSeconds: 0` で送信即終了、タイムアウト指定で返信を待つ。受信側プロンプトは `[Inter-session message ... isUser=false]` 等で**セッション間データ**としてマークされ、受信エージェントはエンドユーザーの直接指示ではなく**ツール経由データ**として扱う必要がある。返信バックループは最大 `session.agentToAgent.maxPingPongTurns`（0–20、既定 5）、`REPLY_SKIP` で早期停止。`sessions_history` は thinking タグ・ツール呼び出し XML・漏れた制御トークンの除去や認証情報のリダクト・切り詰めなど安全性フィルタを適用する。

## 設定・使い方の要点

- **可視性スコープ**：`self`（現セッションのみ）/ `tree`（現＋生成したサブエージェント、**既定**）/ `agent`（このエージェントの全セッション）/ `all`（エージェント間も含む全セッション）。サンドボックス化セッションは設定に関係なく `tree` に制限。
- バイト完全一致が要るならディスク上のトランスクリプトファイルを直接調べる（`sessions_history` は範囲制限ビュー）。

## 注意点・落とし穴

- スレッドスコープのキー（Slack/Discord の `:thread:<id>`）は有効な `sessions_send` ターゲットではない。エージェント間調整は親チャンネルのキーを使い、人間向けスレッドにツール経由メッセージが出ないようにする。
- 完了結果をポーリングで待たない（`sessions_yield` で push を待つ。[[concepts/system-prompt]] の長時間作業ガイダンスも同旨）。

## 用語と略称

- **サブエージェント** = 親から生成される子エージェントセッション
- **オーケストレーション** = 複数セッション/サブエージェントの調整・制御
- **ACP** = Agent Client Protocol（外部ハーネスを動かす規格）
- **リーフ / オーケストレーター** = 末端の子 / 子を管理できる中間の子

## 関連ページ

- [[concepts/session-tool]] — 対応する概念ページ
- [[concepts/session]] / [[concepts/multi-agent]]
- [[concepts/agent]] / [[concepts/agent-loop]]
