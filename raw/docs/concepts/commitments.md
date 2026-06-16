---
title: "推定されたコミットメント"
source: "https://docs.openclaw.ai/ja-JP/concepts/commitments"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
コミットメントは短期間だけ保持されるフォローアップ用メモリです。有効にすると、OpenClaw は 会話から将来のチェックイン機会が生まれたことに気づき、あとで再び取り上げるように 記憶できます。

例:

- 明日の面接について言及した場合。OpenClaw はその後に確認することがあります。
- 疲れ切っていると言った場合。OpenClaw はあとで眠れたかどうか尋ねることがあります。
- エージェントが、何かが変わったあとでフォローアップすると言った場合。OpenClaw はその未完了ループを 追跡することがあります。

コミットメントは `MEMORY.md` のような永続的な事実ではなく、正確な リマインダーでもありません。メモリと自動化の中間に位置します。OpenClaw は 会話に紐づいた義務を記憶し、期限が来ると heartbeat がそれを届けます。

## コミットメントを有効にする

コミットメントはデフォルトでオフです。config で有効にします。

bash

```bash
openclaw config set commitments.enabled true
openclaw config set commitments.maxPerDay 3
```

同等の `openclaw.json`:

json

```json
{
  "commitments": {
    "enabled": true,
    "maxPerDay": 3
  }
}
```

`commitments.maxPerDay` は、ローリング方式の 1 日内でエージェントセッションごとに届けられる 推論されたフォローアップの数を制限します。デフォルトは `3` です。

## 仕組み

エージェントの返信後、OpenClaw は別コンテキストで非表示のバックグラウンド抽出パスを実行することがあります。 このパスは、推論されたフォローアップコミットメントだけを探します。 表示中の会話には書き込まず、メインのエージェントに抽出について 推論させることもありません。

高信頼度の候補が見つかると、OpenClaw は次の情報を持つコミットメントを保存します。

- エージェント ID
- セッションキー
- 元のチャネルと配信先
- 期限ウィンドウ
- 短いチェックイン案
- 送信するかどうかを heartbeat が判断するための、指示ではないメタデータ

配信は heartbeat を通じて行われます。コミットメントの期限が来ると、heartbeat は 同じエージェントおよびチャネルスコープの heartbeat ターンにコミットメントを追加します。 モデルは自然なチェックインを 1 件送信するか、 `HEARTBEAT_OK` と返信して破棄できます。 heartbeat が `target: "none"` で設定されている場合、期限が来たコミットメントは 内部に残り、外部チェックインは送信されません。コミットメント配信プロンプトは 元の会話テキストを再生せず、期限が来たコミットメントの heartbeat ターンは OpenClaw ツールなしで実行されます。

OpenClaw は、推論されたコミットメントを書き込んだ直後に配信することはありません。 期限時刻は、コミットメント作成後少なくとも 1 回分の heartbeat 間隔以降に制限されます。 そのため、フォローアップが推論されたその瞬間にそのまま返ってくることはありません。

## スコープ

コミットメントは、作成された正確なエージェントおよびチャネルコンテキストにスコープされます。 Discord であるエージェントと話している間に推論されたフォローアップは、 別のエージェント、別のチャネル、または無関係なセッションからは配信されません。

このスコープは機能の一部です。自然なチェックインは、グローバルなリマインダーシステムのようではなく、 同じ会話が続いているように感じられるべきです。

## コミットメントとリマインダー

| 必要なこと | 使用するもの |
| --- | --- |
| "Remind me at 3 PM" | [スケジュール済みタスク](https://docs.openclaw.ai/ja-JP/automation/cron-jobs) |
| "Ping me in 20 minutes" | [スケジュール済みタスク](https://docs.openclaw.ai/ja-JP/automation/cron-jobs) |
| "Run this report every weekday" | [スケジュール済みタスク](https://docs.openclaw.ai/ja-JP/automation/cron-jobs) |
| "I have an interview tomorrow" | コミットメント |
| "I was up all night" | コミットメント |
| "Follow up if I do not answer this open thread" | コミットメント |

ユーザーの明示的な依頼は、すでにスケジューラーの経路に属します。コミットメントは 推論されたフォローアップ専用です。つまり、ユーザーがリマインダーを依頼していなくても、 会話の中で将来の有用なチェックインが明確に生まれた場面です。

## コミットメントを管理する

保存されたコミットメントの確認と消去には CLI を使用します。

bash

```bash
openclaw commitments
openclaw commitments --all
openclaw commitments --agent main
openclaw commitments --status snoozed
openclaw commitments dismiss cm_abc123
```

コマンドリファレンスについては [`openclaw commitments`](https://docs.openclaw.ai/ja-JP/cli/commitments) を参照してください。

## プライバシーとコスト

コミットメント抽出は LLM パスを使用するため、有効にすると対象ターンの後にバックグラウンドのモデル 使用量が追加されます。このパスはユーザーに表示される会話からは隠されていますが、 フォローアップが存在するかどうかを判断するために必要な直近のやり取りを読み取ることがあります。

保存されたコミットメントはローカルの OpenClaw 状態です。これは運用上のメモリであり、 長期メモリではありません。この機能は次のコマンドで無効にできます。

bash

```bash
openclaw config set commitments.enabled false
```

## トラブルシューティング

期待したフォローアップが表示されない場合:

- `commitments.enabled` が `true` であることを確認します。
- 保留中、破棄済み、スヌーズ中、または期限切れのレコードについて `openclaw commitments --all` を確認します。
- エージェントで heartbeat が実行されていることを確認します。
- そのエージェントセッションで `commitments.maxPerDay` にすでに達していないか確認します。
- 明示的なリマインダーはコミットメント抽出ではスキップされ、代わりに [スケジュール済みタスク](https://docs.openclaw.ai/ja-JP/automation/cron-jobs) に表示されるべきであることを覚えておいてください。

## 関連

- [メモリ概要](https://docs.openclaw.ai/ja-JP/concepts/memory)
- [Active Memory](https://docs.openclaw.ai/ja-JP/concepts/active-memory)
- [Heartbeat](https://docs.openclaw.ai/ja-JP/gateway/heartbeat)
- [スケジュール済みタスク](https://docs.openclaw.ai/ja-JP/automation/cron-jobs)
- [`openclaw commitments`](https://docs.openclaw.ai/ja-JP/cli/commitments)
- [設定リファレンス](https://docs.openclaw.ai/ja-JP/gateway/configuration-reference#commitments)