---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/sandbox-vs-tool-policy-vs-elevated
source_path: raw/docs/gateway/sandbox-vs-tool-policy-vs-elevated.md
doc_section: gateway
title: "サンドボックス、ツールポリシー、昇格の違い"
ingested: 2026-06-14
tags: [sandbox, tool-policy, elevated, enforcement, disambiguation]
related:
  - "[[concepts/sandboxing]]"
  - "[[concepts/agent]]"
  - "[[concepts/security]]"
---

# サンドボックス、ツールポリシー、昇格の違い（解説）

> 原典: `raw/docs/gateway/sandbox-vs-tool-policy-vs-elevated.md` ・ https://docs.openclaw.ai/ja-JP/gateway/sandbox-vs-tool-policy-vs-elevated
>
> ⚠️ 紛らわしい **3 つの別々の制御**を区別するページ。「ツールがブロックされた」ときどれを直すかの地図。

## 一言まとめ

OpenClaw のツール実行には、関連するが**異なる 3 つの制御**がある：①**サンドボックス**＝どこで実行するか／②**ツールポリシー**＝どのツールを許すか／③**昇格**＝サンドボックス外で実行する exec 専用の脱出口。

## 位置づけ

[[concepts/sandboxing]] の中核的な区別。①は [[concepts/sandboxing]]、②は [[concepts/agent]] のツールポリシー（`tools.*`）、③は tools/elevated。混同すると「サンドボックスを切ればいいのか、ツールを許可すればいいのか、昇格すればいいのか」を取り違える。

## 仕組み・ふるまい

| 制御 | 何を決めるか | 主なキー |
| --- | --- | --- |
| **サンドボックス** | ツールを**どこで**実行するか（サンドボックス内 or ホスト） | `agents.*.sandbox.*`（mode/scope/backend） |
| **ツールポリシー** | **どのツール**を利用可/許可するか | `tools.*`・`tools.sandbox.tools.*`・`agents.list[].tools.*` |
| **昇格** | サンドボックス時に**サンドボックス外で exec する脱出口** | `tools.elevated.*` |

- **ツールポリシー（ハードストップ）**：`deny` が常に優先、`allow` が空でなければ他は全ブロック。`/exec` でも拒否された `exec` は復活しない。**名前でフィルタする**だけで `exec` 内部の副作用は見ない（`exec` を許して `write`/`edit` を拒否してもシェルが読み取り専用になるわけではない）。`group:*`（`runtime`/`fs`/`sessions`/`memory`/`web`/`ui` 等）で複数ツールをまとめて指定。
- **昇格は追加ツールを付与しない**（`exec` にのみ影響）。`/elevated on`（サンドボックス外実行）/`/elevated full`（承認スキップ）。ゲートは `tools.elevated.enabled`＋送信者許可リスト `allowFrom`。Skills スコープでもツール allow/deny の上書きでもない。

## 設定・使い方の要点

- 実態確認：`openclaw sandbox explain [--session agent:main:main|--agent <id>|--json]`（有効モード/スコープ/ツール許可がどこ由来か・昇格ゲート・修正キー）。
- 「ツール X がサンドボックスツールポリシーでブロック」→ `agents.*.sandbox.mode=off`、または `tools.sandbox.tools.deny` から削除/`allow` に追加。
- 「main のはずがサンドボックス化されている」→ `non-main` ではグループ/チャネルキーは main でない。`sandbox explain` の main セッションキーを使うか `mode=off`。

## 注意点・落とし穴

- **バインドマウントはサンドボックス FS を貫通**する（`:ro`/`:rw`）。`/var/run/docker.sock` バインドは実質ホスト制御の付与。`scope: "shared"` はエージェントごとバインドを無視。
- ファイルシステムツールを拒否してもシェル（`exec`）は読み取り専用にならない（読み取り専用が要るなら `group:runtime` も拒否か `workspaceAccess: "ro"`）。

## 用語と略称

- **サンドボックス / ツールポリシー / 昇格** = 実行場所 / 利用可能ツール / exec の脱出口
- **ハードストップ** = 上書きできない強制的な拒否
- **`group:*`** = 複数ツールに展開される省略記法（`group:runtime` 等）
- **脱出口（escape hatch）** = 通常の隔離を意図的に回避する経路

## 関連ページ

- [[concepts/sandboxing]] — 対応する概念ページ（3 制御の地図）
- [[concepts/agent]]（ツールポリシー） / [[concepts/security]]
- [[sources/gateway/sandboxing]] / [[sources/gateway/config-tools]]
