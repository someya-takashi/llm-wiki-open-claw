---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/codex-native-plugins
source_path: raw/docs/plugins/codex-native-plugins.md
doc_section: plugins
title: "ネイティブ Codex プラグイン"
ingested: 2026-06-14
tags: [plugin, codex, native-plugins, migration, app-server, apps]
related:
  - "[[sources/plugins/codex-harness]]"
  - "[[concepts/agent-runtimes]]"
  - "[[components/plugin-system]]"
---

# ネイティブ Codex プラグイン（解説）

> 原典: `raw/docs/plugins/codex-native-plugins.md` ・ https://docs.openclaw.ai/ja-JP/plugins/codex-native-plugins

## 一言まとめ

Codex モードのエージェントが、OpenClaw ターンを処理する**同じ Codex スレッド内で Codex app-server 自身のアプリ/プラグイン**（例 google-calendar）を使えるようにする統合。Plugin 呼び出しはネイティブ Codex トランスクリプトに残り、合成 OpenClaw ツールには変換されない。

## 位置づけ

[[sources/plugins/codex-harness]] の上の機能で、[[concepts/agent-runtimes]] の Codex ランタイム限定（PI/通常 OpenAI/ACP には影響しない）。Plugin 設定は [[components/plugin-system]] の `plugins.entries.codex.config.codexPlugins`。

## 仕組み・ふるまい

- 3 つの独立状態：**インストール済み**（Codex がバンドル保持）／**有効**（OpenClaw 設定が許可）／**アクセス可能**（app-server がアカウントで利用可能と確認）。
- **移行で取り込む**：`openclaw migrate codex --dry-run [--verify-plugin-apps]` → `migrate apply codex --yes`。移行元 Codex home の `openai-curated` プラグインのみ（V1 境界）。ChatGPT サブスクリプションアカウントが必要（非該当は `codex_subscription_required`）。
- **スレッドアプリ設定**：Codex スレッドに制限的 `config.apps` を注入（`_default` 無効、有効な移行済みプラグインのアプリのみ `open_world_enabled: true`）。破壊的操作は `allow_destructive_actions`（既定 true、安全なスキーマのみ自動承認、あいまいな所有権はフェイルクローズ）。

## 設定・使い方の要点

- `plugins.entries.codex.config.codexPlugins`（`enabled`/`allow_destructive_actions`/`plugins.<id>.{marketplaceName,pluginName}`）。変更後は `/new`/`/reset` か Gateway 再起動。
- アプリインベントリは `app/list` を 1 時間メモリキャッシュ（再起動で破棄）。

## 注意点・落とし穴

- トラブルコード：`auth_required`/`app_inaccessible`/`app_disabled`/`app_missing`/`codex_account_unavailable`/`marketplace_missing`/`app_ownership_ambiguous`。表示名のみの一致は所有権が証明されるまで除外。

## 用語と略称

- **app-server** = Codex のローカル実行サーバー
- **アプリ（app）/ プラグイン** = Codex ネイティブの拡張機能
- **openai-curated** = OpenAI 公式のマーケットプレイス
- **elicitation** = ツールがユーザー承認/入力を求める要求
- **fail closed** = あいまい時に拒否する安全側の挙動

## 関連ページ

- [[sources/plugins/codex-harness]] / [[sources/plugins/codex-computer-use]]
- [[concepts/agent-runtimes]] / [[components/plugin-system]]
