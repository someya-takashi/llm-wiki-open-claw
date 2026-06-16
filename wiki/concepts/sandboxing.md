---
type: concept
aliases: [Sandboxing, サンドボックス化, sandbox]
tags: [sandbox, isolation, tool-policy, elevated, security]
related:
  - "[[concepts/security]]"
  - "[[concepts/agent]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/delegate-architecture]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/sandboxing]]"
  - "[[sources/gateway/sandbox-vs-tool-policy-vs-elevated]]"
  - "[[sources/gateway/openshell]]"
updated: 2026-06-14
---

# サンドボックス化

**サンドボックス化**は、エージェントのツール実行（`exec`/`read`/`write`/`edit`/`process`・任意のブラウザ）を**分離環境**で動かして影響範囲を減らす仕組み（任意・既定オフ）。Gateway 本体はホストに残り、有効化したセッションのツールだけがサンドボックスへ入る。

## なぜ重要か

「モデルが誤動作してもファイルシステム/プロセスへのアクセスを構造的に制限する」ための主要な防御。[[concepts/security]] の中核思想「知能より前にアクセス制御」を実装し、[[concepts/multi-agent]]/[[concepts/delegate-architecture]] のエージェントごと境界（Gateway レベル強制）を支える。ただし**完全なセキュリティ境界ではない**――敵対的ローカルユーザーからの分離には別 OS ユーザー/ホストの別 Gateway を使う。

## 3 つの制御を混同しない

ツール実行には**別々の 3 制御**があり、「ブロックされた」ときどれを直すかが変わる（→ [[sources/gateway/sandbox-vs-tool-policy-vs-elevated]]）：

| 制御 | 何を決めるか |
| --- | --- |
| **サンドボックス**（`agents.*.sandbox`） | ツールを**どこで**実行するか |
| **ツールポリシー**（`tools.*`、[[concepts/agent]]） | **どのツール**を許すか（`deny` 優先・ハードストップ） |
| **昇格**（`tools.elevated`） | サンドボックス外で exec する**脱出口**（追加ツールは付与しない） |

## 押さえる点

- **モード**：`off`/`non-main`（グループ/チャネルキーは main でないのでサンドボックス化される＝「意外な挙動」）/`all`。**スコープ**：`agent`/`session`/`shared`。**workspaceAccess**：`none`/`ro`/`rw`。
- **バックエンド**：`docker`（既定）/`ssh`（リモート正本）/`openshell`（管理リモート、mirror/remote → [[sources/gateway/openshell]]）。
- ⚠️ 危険なバインド（`docker.sock`・`/etc`・`~/.ssh` 等）はブロック、`network: host`/`container:*` もブロック。デバッグは `openclaw sandbox explain`。

設定マトリクス・イメージ構築・DooD 制約は [[sources/gateway/sandboxing]]。

コマンド実行とその承認（exec/elevated/exec 承認）を実行時に具体化した面は [[concepts/exec]]。

## 関連

- [[concepts/security]] / [[concepts/agent]]（ツールポリシー） / [[concepts/exec]]
- [[concepts/multi-agent]] / [[concepts/delegate-architecture]] / [[components/gateway]]
