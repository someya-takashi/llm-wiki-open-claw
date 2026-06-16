---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/multi-agent
source_path: raw/docs/concepts/multi-agent.md
doc_section: concepts
title: "マルチエージェントルーティング"
ingested: 2026-06-14
tags: [multi-agent, routing, bindings, agentId, accountId, isolation]
related:
  - "[[concepts/multi-agent]]"
  - "[[concepts/agent]]"
  - "[[concepts/session]]"
---

# マルチエージェントルーティング（解説）

> 原典: `raw/docs/concepts/multi-agent.md` ・ https://docs.openclaw.ai/ja-JP/concepts/multi-agent

## 一言まとめ

1 つの Gateway 内で**複数の分離されたエージェント**を並べて動かし、受信メッセージを**バインディング**で適切なエージェントへ決定的にルーティングする仕組み。各エージェントは独自のワークスペース・状態ディレクトリ（`agentDir`）・セッション履歴・認証プロファイルを持つ。

## 位置づけ

[[concepts/agent]] が定義する「1 プロセスの契約」を**複数並立**させる枠組み。「1 つの Gateway を複数人・複数人格で共有しつつ、頭脳とデータを分離する」のが目的で、[[concepts/session]] の分離（`dmScope`）と表裏一体。組織向けに拡張したのが [[concepts/delegate-architecture]]、並列ワークの設計論が [[concepts/parallel-specialist-lanes]]。

## 仕組み・ふるまい

### エージェントとは（スコープ単位）

完全にスコープ化された「頭脳」で、独自の **ワークスペース**（`AGENTS.md`/`SOUL.md`/`USER.md` 等）・**状態ディレクトリ `agentDir`**（`~/.openclaw/agents/<agentId>/`、認証プロファイル・モデルレジストリ）・**セッションストア**（`.../sessions`）を持つ。認証プロファイル（`auth-profiles.json`）は**エージェントごと**。

### 主要な概念

- `agentId`：1 つの頭脳（既定 `main`、セッションは `agent:main:<mainKey>`）。
- `accountId`：1 つのチャネルアカウントインスタンス（例 WhatsApp の `personal`/`biz`）。
- `binding`：`(channel, accountId, peer)` ＋任意の guild/team ID で受信メッセージを `agentId` にマッピング。

### ルーティングルール（決定的・最も具体的なものが優先）

優先順は **peer マッチ → parentPeer（スレッド継承）→ guildId+roles → guildId → teamId → チャネルの accountId → チャネルレベル（`accountId:"*"`）→ デフォルトエージェント**。同階層で複数マッチなら設定順で最初、複数マッチフィールドは AND セマンティクス。

## 設定・使い方の要点

- 追加：`openclaw agents add work`、確認：`openclaw agents list --bindings`。`agents.list` にエージェント、`channels.<ch>.accounts` にアカウント、`bindings` で接続。
- **エージェントごとのサンドボックス/ツール**：`agents.list[].sandbox`・`agents.list[].tools.allow/deny` で境界を Gateway レベル強制（人格ファイルとは独立）。`tools.elevated` はグローバルでエージェント単位にできない。
- 1 WhatsApp 番号で DM 分割：`peer.kind: "direct"` で送信者 E.164 をマッチ（ただし DM はメインセッションキーに集約されるため真の分離には 1 人 1 エージェント）。
- クロスエージェント QMD 検索：`agents.list[].memorySearch.qmd.extraCollections`。

## 注意点・落とし穴

- **`agentDir` を複数エージェントで再利用しない**（認証/セッション衝突）。ローカルプロファイルが無ければデフォルトエージェントの認証を参照できるが、OAuth リフレッシュトークンはセカンダリへ複製されない（独立アカウントが要るなら各自サインイン）。
- DM アクセス制御は**エージェントごとでなく WhatsApp アカウントごとにグローバル**。
- ワークスペースは既定 cwd であって強制サンドボックスではない（→ [[concepts/sandboxing]]）。

## 用語と略称

- **agentId / accountId / binding** = 頭脳 / チャネルアカウント / ルーティング規則
- **agentDir** = エージェントごとの状態ディレクトリ（認証・モデル・設定）
- **E.164** = 国際電話番号の標準形式（`+15551234567`）
- **AND セマンティクス** = 指定した全マッチフィールドを満たす必要がある

## 関連ページ

- [[concepts/multi-agent]] — 対応する概念ページ
- [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]]
- [[concepts/agent]] / [[concepts/session]] / [[concepts/session-tool]]
- [[concepts/sandboxing]] / [[concepts/channel-routing]]
