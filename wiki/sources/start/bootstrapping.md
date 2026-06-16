---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/start/bootstrapping
source_path: raw/docs/start/bootstrapping.md
doc_section: start
title: "エージェントのブートストラップ"
ingested: 2026-06-14
tags: [bootstrapping, onboarding, workspace, first-run, identity]
related:
  - "[[concepts/agent-workspace]]"
  - "[[concepts/agent]]"
  - "[[concepts/system-prompt]]"
---

# エージェントのブートストラップ（解説）

> 原典: `raw/docs/start/bootstrapping.md` ・ https://docs.openclaw.ai/ja-JP/start/bootstrapping

## 一言まとめ

ブートストラップは、エージェントのワークスペースを準備し ID 詳細を収集する**初回実行の手順**。オンボーディングの後、エージェントが初めて起動するときに 1 回だけ実行される。

## 位置づけ

`start/` セクション（導入・セットアップ）の 1 ページで、[[concepts/agent-workspace]] の標準ファイルを実際に生成する初回フロー。生成された `IDENTITY.md`/`USER.md`/`SOUL.md` は以後 [[concepts/system-prompt]] によって毎ターン注入される。[[concepts/agent]] が前提とする「ワークスペースが整っている」状態を作る工程に当たる。

## 仕組み・ふるまい

- 初回実行時、OpenClaw はワークスペース（既定 `~/.openclaw/workspace`）をブートストラップする：
  - `AGENTS.md`・`BOOTSTRAP.md`・`IDENTITY.md`・`USER.md` をシードする。
  - 短い Q&A 手順（**1 回に 1 つの質問**）を実行する。
  - ID と設定を `IDENTITY.md`・`USER.md`・`SOUL.md` に書き込む。
  - 完了時に `BOOTSTRAP.md` を削除し、1 回だけ実行されるようにする。
- 埋め込み/ローカルモデルで実行する場合、`BOOTSTRAP.md` は特権付きシステムコンテキストには含めず、`read` ツールを確実に呼ばないモデルでも完了できるよう**ユーザープロンプト内でファイル内容を渡す**。ワークスペースに安全にアクセスできない実行では、汎用挨拶ではなく限定的なブートストラップノートを受け取る。

## 設定・使い方の要点

- 事前シード済みワークスペースでスキップ：`openclaw onboard --skip-bootstrap`。
- **実行場所は常に Gateway ホスト**。macOS アプリがリモート Gateway に接続している場合、ワークスペースとブートストラップファイルはそのリモートマシン上にある（例 `user@gateway-host:~/.openclaw/workspace` を編集）。

## 注意点・落とし穴

- ブートストラップは「新しいワークスペース」を前提とする一度きりの儀式。既存ワークスペースでは想定どおり動かないことがある（[[concepts/agent]] の `skipBootstrap` も参照）。

## 用語と略称

- **ブートストラップ** = 初回実行でワークスペースと ID を準備する手順
- **オンボーディング** = ブートストラップの前段にあたる初期セットアップ（macOS アプリ/CLI）

## 関連ページ

- [[concepts/agent-workspace]] — 生成されるファイルのレイアウト
- [[concepts/agent]] / [[concepts/system-prompt]]
