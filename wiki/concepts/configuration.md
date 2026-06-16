---
type: concept
aliases: [Configuration, 設定, openclaw.json]
tags: [configuration, json5, hot-reload, schema, secrets]
related:
  - "[[concepts/architecture]]"
  - "[[concepts/agent]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/authentication]]"
  - "[[concepts/secrets]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/configuration]]"
  - "[[sources/gateway/configuration-reference]]"
  - "[[sources/gateway/configuration-examples]]"
  - "[[sources/gateway/config-agents]]"
  - "[[sources/gateway/config-channels]]"
  - "[[sources/gateway/config-tools]]"
updated: 2026-06-14
---

# 設定（Configuration）

OpenClaw の挙動は、単一の **`~/.openclaw/openclaw.json`（JSON5）** で宣言的に設定する。チャネル・モデル・ツール・サンドボックス・自動化・セッションなど、ほぼすべての振る舞いがこのファイルに集約され、[[components/gateway]] が監視して反映する。

## なぜ重要か

「設定を 1 ファイルに集約し、厳格に検証し、ホットリロードする」という設計が、OpenClaw の運用安全性の土台になっている。**スキーマに完全一致しない設定は Gateway が起動を拒否**（不明キー・型不正は即エラー）するため、壊れた設定が静かに動き続けることがない。`gateway.*`/`plugins` 以外はほぼ**再起動なしでホット適用**されるため、運用中の調整が軽い。エージェント自身も `config.schema.lookup`→`config.get`→`config.patch` という RPC で安全に自己設定できる（[[components/gateway]] の `gateway` ツール）。

## 押さえる点

- **JSON5**（コメント・末尾カンマ可）。編集手段は `openclaw onboard`/`configure`、`config get/set/unset`、Control UI、直接編集。
- **厳格検証**：失敗時は診断コマンドのみ動作、`openclaw doctor --fix` で修復。
- **ホットリロード**（既定 `hybrid`）：安全な変更はホット適用、`gateway.*`・`discovery`・`plugins` のみ再起動。
- **`$include`** で複数ファイル分割、**`${VAR}`**／**SecretRef**（env/file/exec、→ [[concepts/secrets]]）で秘密情報を参照。接続/モデルの認証は [[concepts/authentication]]。
- ドメイン別の詳細：[[sources/gateway/config-agents]]（agents/session/messages）・[[sources/gateway/config-channels]]（channels）・[[sources/gateway/config-tools]]（tools/カスタムプロバイダー）。全フィールド辞書は [[sources/gateway/configuration-reference]]、コピペ例は [[sources/gateway/configuration-examples]]。

タスク指向のガイドは [[sources/gateway/configuration]]、Gateway の起動・運用は [[sources/gateway/gateway]]。

## 関連

- [[components/gateway]] / [[concepts/architecture]]
- [[concepts/agent]] / [[concepts/multi-agent]] / [[concepts/session]] / [[concepts/sandboxing]]
