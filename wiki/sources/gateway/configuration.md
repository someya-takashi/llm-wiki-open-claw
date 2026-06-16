---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/configuration
source_path: raw/docs/gateway/configuration.md
doc_section: gateway
title: "設定"
ingested: 2026-06-14
tags: [configuration, json5, hot-reload, include, config-rpc, secrets]
related:
  - "[[concepts/configuration]]"
  - "[[components/gateway]]"
  - "[[sources/gateway/configuration-reference]]"
---

# 設定（解説）

> 原典: `raw/docs/gateway/configuration.md` ・ https://docs.openclaw.ai/ja-JP/gateway/configuration

## 一言まとめ

OpenClaw の設定は `~/.openclaw/openclaw.json` の **JSON5**（コメント可・末尾カンマ可の緩い JSON）で行う、というタスク指向のセットアップガイド。チャネル接続・モデル・サンドボックス・自動化・セッションなど「よくある設定タスク」のレシピ集。

## 位置づけ

[[concepts/configuration]] の実践面。網羅的なフィールド一覧は [[sources/gateway/configuration-reference]]、コピペ例は [[sources/gateway/configuration-examples]]、ドメイン別の詳細は [[sources/gateway/config-agents]]/[[sources/gateway/config-channels]]/[[sources/gateway/config-tools]]。

## 仕組み・ふるまい

### 編集手段

- **対話型**：`openclaw onboard`（フル）/`openclaw configure`（ウィザード）。
- **CLI ワンライナー**：`openclaw config get/set/unset <path> [value]`。
- **Control UI**：`http://127.0.0.1:18789` の Config タブ（ライブスキーマからフォーム描画＋Raw JSON エディタ）。
- **直接編集**：`openclaw.json` を書く（Gateway が監視して自動適用）。

### 厳格な検証

⚠️ **スキーマに完全一致する設定のみ受理**。不明なキー・型不正・無効値があると **Gateway は起動を拒否**（ルートの例外は `$schema` 文字列のみ）。検証失敗時は診断コマンド（`doctor`/`logs`/`health`/`status`）のみ動作。`openclaw doctor --fix` で修復、または「最後に正常だったコピー」へ復元（自動復元はしない）。

### ホットリロード

Gateway は `openclaw.json` を監視して自動適用。`gateway.reload.mode`：**`hybrid`（既定）**＝安全な変更はホット適用・重要な変更は自動再起動／`hot`／`restart`／`off`。**再起動が必要なのは `gateway.*`（port/bind/auth/TLS/HTTP）と `discovery`/`plugins` のみ**で、`channels`/`agents`/`models`/`hooks`/`cron`/`session`/`messages`/`tools`/`ui` 等はホット適用。

### $include（複数ファイル分割）

`{ agents: { $include: "./agents.json5" } }`。単一ファイルは置換、配列は順次ディープマージ、兄弟キーは include 後にマージ（上書き）、ネスト最大 10 段、相対パス解決、`openclaw.json` のディレクトリ配下に閉じ込め（`OPENCLAW_INCLUDE_ROOTS` で追加許可）。OpenClaw 所有の書き込みは、単一トップレベルセクションが単一ファイル include なら include 先を更新する。

## 設定・使い方の要点

- **最小設定**例：`{ agents: { defaults: { workspace: "~/.openclaw/workspace" } }, channels: { whatsapp: { allowFrom: ["+1555..."] } } }`。
- **設定 RPC**（プログラム更新）：`config.schema.lookup`（1 サブツリーのスキーマ＋子概要）→ `config.get`（スナップショット＋`hash`）→ `config.patch`（JSON マージパッチ。オブジェクトはマージ・`null` 削除・配列置換）→ 全置換は `config.apply`、自己更新＋再起動は `update.run`。`config.apply`/`patch` は既存設定時 `baseHash` 必須。コントロールプレーン書き込みは deviceId+clientIp ごと 60 秒 3 リクエストにレート制限。
- **環境変数**：`.env`（cwd）/`~/.openclaw/.env`（既存 env は上書きしない）。設定内 `env.vars`、文字列値内 `${VAR_NAME}`（大文字のみ、未設定はエラー、`$${VAR}` でエスケープ）。**SecretRef**（`{ source: "env"|"file"|"exec", provider, id }`）で API キー等を参照（→ gateway/secrets）。
- **エージェント/自動化は編集前に `config.schema.lookup` を引く**（正確なフィールド定義・制約の一次情報源）。

## 注意点・落とし穴

- アクティブな設定パスは**通常ファイル**であること（シンボリックリンクは OpenClaw 所有書き込みで非対応。アトミック書き込みで置換され得る）。デフォルト外なら `OPENCLAW_CONFIG_PATH` を実ファイルに向ける。
- 直接編集は検証されるまで信頼されない。`gateway.mode` 削除やファイルサイズ半減のような破壊的上書きは拒否され `.rejected.*` に保存。
- `***` のような秘匿プレースホルダーを含む候補は「最後に正常だったコピー」へ昇格しない。

## 用語と略称

- **JSON5** = コメント・末尾カンマ・引用符省略を許す緩い JSON 方言
- **ホットリロード** = 再起動せず設定変更を反映すること
- **$include** = 設定を別ファイルに分割・合成する仕組み
- **SecretRef** = 秘密情報を env/file/exec から参照するオブジェクト
- **JSON マージパッチ** = 部分更新規約（オブジェクト=マージ / null=削除 / 配列=置換）

## 関連ページ

- [[concepts/configuration]] — 対応する概念ページ
- [[sources/gateway/configuration-reference]] / [[sources/gateway/configuration-examples]]
- [[sources/gateway/config-agents]] / [[sources/gateway/config-channels]] / [[sources/gateway/config-tools]]
- [[components/gateway]]
