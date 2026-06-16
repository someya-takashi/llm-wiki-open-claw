---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/local-model-services
source_path: raw/docs/gateway/local-model-services.md
doc_section: gateway
title: "ローカルモデルサービス"
ingested: 2026-06-14
tags: [local-models, local-service, on-demand, process-management, openai-compat]
related:
  - "[[concepts/local-models]]"
  - "[[sources/gateway/local-models]]"
  - "[[components/gateway]]"
---

# ローカルモデルサービス（解説）

> 原典: `raw/docs/gateway/local-model-services.md` ・ https://docs.openclaw.ai/ja-JP/gateway/local-model-services

## 一言まとめ

`models.providers.<id>.localService` を使うと、OpenClaw が**選んだモデルが必要になったときだけローカルモデルサーバーを起動**し、準備完了を待ってからリクエストを送る。常時起動はコスト高なローカルサーバー向けの「オンデマンド起動」。

## 位置づけ

[[concepts/local-models]] の運用補助。[[sources/gateway/local-models]] でプロバイダーを登録した上で、そのサーバープロセスの**ライフサイクル管理**を担う。

## 仕組み・ふるまい

1. モデルリクエストがプロバイダーに解決 → 2. `localService` があれば `healthUrl` をプローブ → 3. 成功なら既存サーバーを再利用 → 4. 失敗なら `command`＋`args` を起動 → 5. `readyTimeoutMs` まで準備完了をポーリング → 6. 通常トランスポートで送信 → 7. OpenClaw が起動したプロセスは `idleStopMs`（>0 時）アイドル後に停止。

- ⚠️ サーバーは **launchd/systemd/Docker を入れず、最初に必要とした OpenClaw プロセスの子プロセス**として起動。同じ `healthUrl` が既に稼働していれば引き継がず再利用。
- 起動は「プロバイダーの command+args 組」ごとに**直列化**（同時リクエストで重複サーバーを作らない）。アクティブなストリーミング応答はリースを保持し、アイドル停止は本文処理完了まで待つ。

## 設定・使い方の要点

- フィールド：`command`（絶対パス・シェル検索なし）・`args`（シェル展開なし）・`cwd`・`env`（OpenClaw 環境にマージ）・`healthUrl`（省略時は `baseUrl`＋`/models`）・`readyTimeoutMs`（既定 120000）・`idleStopMs`（`0`/省略で OpenClaw 終了まで起動継続）。
- 低速プロバイダーには `timeoutSeconds` を上げてコールドスタートがタイムアウトに当たらないように。例として `inferrs`（`/opt/homebrew/bin/inferrs serve ... --device metal`）・`ds4`（`.gguf` を `--ctx 393216` で）。

## 注意点・落とし穴

- `command` は絶対パス（`which <bin>` の結果に置き換える）。`/v1/models` 以外で準備完了を公開するサーバーには明示 `healthUrl`。

## 用語と略称

- **localService** = プロバイダー所有のローカルサーバーをオンデマンド起動する設定
- **healthUrl** = 起動完了を判定するプローブ先
- **GGUF** = ローカル推論のモデル重みフォーマット
- **リース（lease）** = 実行中ストリームがプロセス停止を保留する仕組み

## 関連ページ

- [[concepts/local-models]] / [[sources/gateway/local-models]]
- [[sources/gateway/cli-backends]] / [[components/gateway]]
