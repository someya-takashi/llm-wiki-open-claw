---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/skills-config
source_path: raw/docs/tools/skills-config.md
doc_section: tools
title: "Skills 設定"
ingested: 2026-06-14
tags: [skills, config, allowlist, watch, install, sandbox, secretref]
related:
  - "[[concepts/skills]]"
  - "[[sources/tools/skills]]"
  - "[[concepts/configuration]]"
---

# Skills 設定（解説）

> 原典: `raw/docs/tools/skills-config.md` ・ https://docs.openclaw.ai/ja-JP/tools/skills-config

## 一言まとめ

`skills.*`（ローダー/インストール設定）と `agents.*.skills`（エージェントごとの可視性）の設定スキーマと例。Skill の読み込み・許可リスト・監視・インストーラ・シークレット注入をまとめて宣言する。

## 位置づけ

[[concepts/skills]] の設定面（概念は [[sources/tools/skills]]）。[[concepts/configuration]] の `skills` ブロックで、秘密値は [[concepts/secrets]]（SecretRef）。

## 仕組み・ふるまい

- `skills.allowBundled`（バンドル skill 専用許可リスト）・`skills.load`（`extraDirs`/`allowSymlinkTargets`/`watch`/`watchDebounceMs`）・`skills.install`（`preferBrew`/`nodeManager` npm|pnpm|yarn|bun/`allowUploadedArchives`）・`skills.entries.<key>`（`enabled`/`env`/`apiKey`）。
- 許可リスト：`agents.defaults.skills`（ベースライン）と `agents.list[].skills`（**置換**、`[]` で無効）。読み込み優先順位は workspace→.agents→~/.openclaw→bundled→extraDirs。

## 設定・使い方の要点

- `entries.<skillKey>.apiKey` はプレーン or SecretRef（`{source,provider,id}`）。キーは既定で skill 名（`metadata.openclaw.skillKey` があればそれ）。
- 組み込み画像生成は skill でなくコアの `image_generate`＋`agents.defaults.imageGenerationModel` を優先。

## 注意点・落とし穴

- ⚠️ **サンドボックス化セッションではホストの `env`/`apiKey` が効かない**（サンドボックスは `process.env` を継承しない）。`agents.defaults.sandbox.docker.env` かカスタムイメージに env を入れる。さもないと `apiKey not configured` で失敗。
- シンボリックリンクは既定でルート外を拒否（`Skipping escaped skill path`）。`allowSymlinkTargets` は狭く保つ（広いルートを指さない）。

## 用語と略称

- **allowlist（許可リスト）** = エージェントに見せる skill の集合
- **ウォッチャー（watcher）** = `SKILL.md` 変更を検知してホットリロードする監視
- **SecretRef** = env/file/exec から秘密を解決する参照（[[concepts/secrets]]）
- **nodeManager** = skill インストールに使う Node パッケージマネージャ

## 関連ページ

- [[concepts/skills]] / [[sources/tools/skills]] / [[sources/tools/creating-skills]]
- [[concepts/configuration]] / [[concepts/secrets]] / [[concepts/sandboxing]]
