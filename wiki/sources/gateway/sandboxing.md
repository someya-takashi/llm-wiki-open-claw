---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/sandboxing
source_path: raw/docs/gateway/sandboxing.md
doc_section: gateway
title: "サンドボックス化"
ingested: 2026-06-14
tags: [sandbox, isolation, docker, ssh, openshell, tool-execution]
related:
  - "[[concepts/sandboxing]]"
  - "[[concepts/security]]"
  - "[[concepts/multi-agent]]"
---

# サンドボックス化（解説）

> 原典: `raw/docs/gateway/sandboxing.md` ・ https://docs.openclaw.ai/ja-JP/gateway/sandboxing

## 一言まとめ

OpenClaw は影響範囲を減らすため、**ツール実行をサンドボックスバックエンド内で**動かせる（任意・既定オフ）。Gateway 本体はホストに残り、有効化されたセッションのツール（`exec`/`read`/`write`/`edit`/`process`・任意のブラウザ）が分離環境で走る。

## 位置づけ

[[concepts/sandboxing]] の本体。[[concepts/security]] のハードニング手段であり、[[concepts/multi-agent]] のエージェントごと境界とも組み合わさる。「どこで実行するか（サンドボックス）」は、「どのツールを許すか（ツールポリシー）」「サンドボックス外で走らせる脱出口（昇格）」と別物――その区別は [[sources/gateway/sandbox-vs-tool-policy-vs-elevated]]。

## 仕組み・ふるまい

- **モード**（`agents.defaults.sandbox.mode`）：`off`／`non-main`（**non-main セッションのみ**＝グループ/チャネルキーは main でないのでサンドボックス化される。よくある「意外な挙動」）／`all`。
- **スコープ**（`scope`）：`agent`（既定、エージェント 1 コンテナ）／`session`／`shared`。
- **バックエンド**（`backend`）：`docker`（既定、ローカル `/var/run/docker.sock`）／`ssh`（任意の SSH ホスト、リモート正本）／`openshell`（管理リモート、→ [[sources/gateway/openshell]]）。
- **ワークスペースアクセス**（`workspaceAccess`）：`none`（既定、サンドボックス内ワークスペースのみ）／`ro`（`/agent` に読み取り専用マウント）／`rw`（`/workspace` に読み書き）。
- **サンドボックス化されないもの**：Gateway プロセス本体、`tools.elevated` で明示的にサンドボックス外実行を許したツール（昇格 exec はサンドボックスを迂回し既定 `gateway`、exec 対象が `node` なら `node`）。

## 設定・使い方の要点

- **イメージ**：既定 `openclaw-sandbox:bookworm-slim`（要ビルド、`scripts/sandbox-setup.sh` か `docker build`。存在しないと即失敗＝勝手に `debian` で代替しない。書き込み/編集ヘルパー用に `python3` を含む）。既定でネットワーク**なし**（`docker.network` で上書き）。
- **バインドマウント**（`docker.binds`、`host:container:mode`）：グローバルとエージェントごとはマージ（`shared` ではエージェント分は無視）。⚠️ ソース/シークレットは `:ro` 優先。
- **`setupCommand`**：コンテナ作成後に 1 回だけ実行（ネットワーク・root・書き込み可ルートが要る）。サンドボックス exec はホストの `process.env` を継承しない（`sandbox.docker.env` で渡す）。
- デバッグ：`openclaw sandbox explain [--session|--agent|--json]`（有効モード/スコープ/ツール許可・昇格ゲート・修正キー）。

## 注意点・落とし穴

- **完全なセキュリティ境界ではない**（モデルの誤動作に対する実質的なファイル/プロセス制限）。敵対的ローカルユーザーからの分離には別 OS ユーザー/ホストの別 Gateway を使う。
- ⚠️ **危険なバインドはブロック**：`docker.sock`・`/etc`・`/proc`・`/sys`・`/dev`、`~/.ssh`/`~/.aws`/`~/.gnupg` 等の認証情報ルート。検証はシンボリックリンク親脱出も閉じる（最深の既存祖先まで解決して再チェック）。`network: "host"`/`container:*` はブロック（緊急回避は `dangerouslyAllowContainerNamespaceJoin`）。
- **ツールポリシーが先に効く**：拒否されたツールはサンドボックス化しても復活しない（[[sources/gateway/sandbox-vs-tool-policy-vs-elevated]]）。
- **DooD（Docker-out-of-Docker）**：Gateway をコンテナ化する場合、`workspace` はホスト絶対パス＋同一ボリュームマップ（`-v /home/user/.openclaw:/home/user/.openclaw`）が必要（さもないと Heartbeat 書き込みで `EACCES`）。

## 用語と略称

- **サンドボックス** = ツール実行を隔離する分離環境（コンテナ/リモート）
- **バックエンド** = サンドボックスを提供するランタイム（docker/ssh/openshell）
- **scope** = 作るコンテナ数の単位（agent/session/shared）
- **workspaceAccess** = サンドボックスがワークスペースを見られる範囲（none/ro/rw）
- **DooD** = Docker-out-of-Docker（ホスト Docker ソケットで兄弟コンテナを起動）
- **CDP** = Chrome DevTools Protocol（サンドボックスブラウザ制御）

## 関連ページ

- [[concepts/sandboxing]] — 対応する概念ページ
- [[sources/gateway/sandbox-vs-tool-policy-vs-elevated]] / [[sources/gateway/openshell]]
- [[concepts/security]] / [[concepts/multi-agent]] / [[sources/gateway/config-agents]]
