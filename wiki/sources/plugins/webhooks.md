---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/webhooks
source_path: raw/docs/plugins/webhooks.md
doc_section: plugins
title: "Webhook Plugin"
ingested: 2026-06-14
tags: [plugin, webhooks, taskflow, automation, ingress, secret]
related:
  - "[[components/plugin-system]]"
  - "[[components/gateway]]"
  - "[[concepts/secrets]]"
---

# Webhook Plugin（解説）

> 原典: `raw/docs/plugins/webhooks.md` ・ https://docs.openclaw.ai/ja-JP/plugins/webhooks

## 一言まとめ

外部オートメーション（Zapier・n8n・CI・内部サービス）を OpenClaw の **TaskFlow に結ぶ認証付き HTTP 入口**を追加する Plugin。カスタム Plugin を書かずに、信頼済みシステムから管理対象 TaskFlow を作成・進行できる。

## 位置づけ

[[components/plugin-system]] の入口系 Plugin で、[[components/gateway]] のプロセス内で実行（同じ HTTP サーバー上に Webhook ルート）。共有シークレットは [[concepts/secrets]]（SecretRef）で外部化推奨。

## 仕組み・ふるまい

- `plugins.entries.webhooks.config.routes.<id>`：`path`（既定 `/plugins/webhooks/<id>`）・`sessionKey`（バインド先 TaskFlow を所有・必須）・`secret`（必須、プレーン or SecretRef `source: env|file|exec`）・`controllerId`・`description`。
- リクエストは `POST`＋`Authorization: Bearer <secret>`（or `x-openclaw-webhook-secret`）。`action` で `create_flow`/`get_flow`/`list_flows`/`resume_flow`/`finish_flow`/`fail_flow`/`run_task` 等を呼ぶ。
- レスポンスは `{ok, routeId, result}`（拒否は `code`/`error`）。所有者/セッションメタデータは意図的に除去。

## 設定・使い方の要点

- セキュリティ：各ルートは設定 `sessionKey` の TaskFlow 権限で動く。**ルートごとに強い一意シークレット**・SecretRef 優先・最も狭いセッションにバインド・必要な path だけ公開。Plugin は共有シークレット認証・本文サイズ/タイムアウトガード・固定窓レート制限・実行中リクエスト制限を適用。
- シークレット未解決のルートは壊れた口を出さずスキップ（警告ログ）。

## 注意点・落とし穴

- リモート Gateway では Gateway ホストに Plugin をインストール・設定して再起動。`run_task` の `runtime`（例 `acp`）と `childSessionKey` で子タスクを作る。

## 用語と略称

- **TaskFlow** = OpenClaw の管理対象タスク/フロー（[[concepts/queue]] のタスク台帳系）
- **イングレス（ingress）** = 外部からの受け口
- **SecretRef** = env/file/exec から秘密を解決する参照（[[concepts/secrets]]）
- **共有シークレット** = ルート認証に使う事前共有の秘密

## 関連ページ

- [[components/plugin-system]] / [[components/gateway]] / [[concepts/secrets]]
- [[concepts/queue]] / [[concepts/heartbeat]]
