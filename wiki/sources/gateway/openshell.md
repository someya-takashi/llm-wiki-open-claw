---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/openshell
source_path: raw/docs/gateway/openshell.md
doc_section: gateway
title: "OpenShell"
ingested: 2026-06-14
tags: [sandbox, openshell, backend, ssh, remote, plugin, workspace-mode]
related:
  - "[[concepts/sandboxing]]"
  - "[[sources/gateway/sandboxing]]"
---

# OpenShell（解説）

> 原典: `raw/docs/gateway/openshell.md` ・ https://docs.openclaw.ai/ja-JP/gateway/openshell

## 一言まとめ

OpenShell は OpenClaw の**マネージドサンドボックスバックエンド**。ローカル Docker の代わりに、`openshell` CLI にサンドボックスのライフサイクルを委譲し、**SSH ベースのリモート環境**をプロビジョニングする。

## 位置づけ

[[concepts/sandboxing]] のバックエンドの 1 つ（`backend: "openshell"`）。汎用 SSH バックエンドと同じトランスポート/FS ブリッジを再利用しつつ、OpenShell 固有のライフサイクル（`sandbox create/get/delete`/`ssh-config`）と任意の `mirror` ワークスペースモードを足す。比較は [[sources/gateway/sandboxing]] のバックエンド表。

## 仕組み・ふるまい

- **ワークスペースモード**（最重要の判断、`plugins.entries.openshell.config.mode`）：
  - **`mirror`（既定）**：ローカルワークスペースが正本。exec 前にローカル→リモート同期、exec 後にリモート→ローカル同期（双方向）。Docker バックエンドに近い挙動。ターンごとの同期コストが高い。
  - **`remote`**：OpenShell ワークスペースが正本。作成時に 1 回シードし、以後 `exec`/`read`/`write`/`edit` はリモートに直接（同期で戻さない）。オーバーヘッド低。長時間エージェント/CI 向け。⚠️ シード後にホスト側で編集してもリモートは認識しない（`openclaw sandbox recreate` で再シード）。
- **動作フロー**：`openshell sandbox create`（`--from`/`--gateway`/`--policy`/`--providers`/`--gpu`）→ `openshell sandbox ssh-config <name>` で SSH 詳細取得 → SSH 設定を一時ファイルに書き、汎用 SSH と同じ FS ブリッジでセッションを開く。

## 設定・使い方の要点

- 前提：`openshell` CLI が `PATH` 上（または `config.command`）、OpenShell アカウント、Gateway 稼働。
- サンドボックス側は他バックエンドと同じく `agents.defaults.sandbox`（`mode`/`scope`/`workspaceAccess`/`backend: "openshell"`）。OpenShell 固有は `plugins.entries.openshell.config`（`mode`/`from`/`gateway`/`gatewayEndpoint`/`policy`/`providers`/`gpu`/`remoteWorkspaceDir`〔既定 `/sandbox`〕/`remoteAgentWorkspaceDir`〔`/agent`〕/`timeoutSeconds`〔120〕）。
- ライフサイクル：`openclaw sandbox list`（Docker＋OpenShell）/`explain`/`recreate --all`。`backend`/`from`/`mode`/`policy` を変えたら **recreate** が必要。

## 注意点・落とし穴

- 現在の制限：**サンドボックスブラウザ非対応**、`sandbox.docker.binds` と `docker.*` ノブは適用されない（Docker 専用）。
- セキュリティ強化：ワークスペースのルート fd を固定し、各読み取り前にサンドボックス ID を再確認（シンボリックリンク差し替え・再マウントによる読み取りリダイレクトを防ぐ）。

## 用語と略称

- **マネージドバックエンド** = ライフサイクルを外部 CLI に委譲するサンドボックス
- **mirror / remote** = ローカル正本（双方向同期）/ リモート正本（1 回シード）
- **正本（canonical）ワークスペース** = 真実のソースとなる側
- **シード（seed）** = 初回にローカルからリモートへ内容を流し込むこと

## 関連ページ

- [[concepts/sandboxing]] — 対応する概念ページ
- [[sources/gateway/sandboxing]] — バックエンド比較・全体
