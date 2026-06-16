---
title: "エージェントのブートストラップ"
source: "https://docs.openclaw.ai/ja-JP/start/bootstrapping"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
ブートストラップは、エージェントのワークスペースを準備し、ID 詳細を収集する **初回実行** の手順です。これはオンボーディングの後、エージェントが初めて起動するときに実行されます。

## ブートストラップで行われること

エージェントの初回実行時に、OpenClaw はワークスペース（デフォルトは `~/.openclaw/workspace` ）をブートストラップします。

- `AGENTS.md` 、 `BOOTSTRAP.md` 、 `IDENTITY.md` 、 `USER.md` をシードします。
- 短い Q&A 手順を実行します（1 回に 1 つの質問）。
- ID と設定を `IDENTITY.md` 、 `USER.md` 、 `SOUL.md` に書き込みます。
- 完了時に `BOOTSTRAP.md` を削除し、1 回だけ実行されるようにします。

埋め込みモデルまたはローカルモデルで実行する場合、OpenClaw は `BOOTSTRAP.md` を特権付きシステムコンテキストに含めません。主要な対話型の初回実行では、 `read` ツールを確実には呼び出さないモデルでもこの手順を完了できるように、ユーザープロンプト内でファイルの内容を引き続き渡します。現在の実行がワークスペースに安全にアクセスできない場合、エージェントは汎用的な挨拶ではなく、限定的なブートストラップノートを受け取ります。

## ブートストラップをスキップする

事前にシード済みのワークスペースでこれをスキップするには、 `openclaw onboard --skip-bootstrap` を実行します。

## 実行される場所

ブートストラップは常に **Gateway ホスト** で実行されます。macOS アプリがリモート Gateway に接続している場合、ワークスペースとブートストラップファイルはそのリモートマシン上にあります。

> [!note] Note
> **Note**
> 
> Gateway が別のマシンで実行されている場合は、Gateway ホスト上のワークスペースファイルを編集してください（例: `user@gateway-host:~/.openclaw/workspace` ）。

## 関連ドキュメント

- macOS アプリのオンボーディング: [オンボーディング](https://docs.openclaw.ai/ja-JP/start/onboarding)
- ワークスペースレイアウト: [エージェントワークスペース](https://docs.openclaw.ai/ja-JP/concepts/agent-workspace)