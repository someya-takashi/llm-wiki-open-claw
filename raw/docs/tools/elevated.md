---
title: "昇格モード"
source: "https://docs.openclaw.ai/ja-JP/tools/elevated"
author:
published:
created: 2026-06-14
description: "昇格 exec モード: サンドボックス化されたエージェントからサンドボックス外でコマンドを実行する"
tags:
  - "clippings"
---
エージェントがサンドボックス内で実行されると、その `exec` コマンドは サンドボックス環境に閉じ込められます。 **昇格モード** により、エージェントは代わりにサンドボックスの外へ出て コマンドを実行でき、承認ゲートも設定できます。

> [!note] Note
> **Info**
> 
> 昇格モードは、エージェントが **サンドボックス化** されている場合にのみ動作を変更します。サンドボックス化されていないエージェントでは、exec はすでにホスト上で実行されます。

## ディレクティブ

スラッシュコマンドでセッションごとに昇格モードを制御します。

| ディレクティブ | 動作 |
| --- | --- |
| `/elevated on` | 設定されたホストパス上でサンドボックスの外で実行し、承認は維持します |
| `/elevated ask` | `on` と同じ（エイリアス） |
| `/elevated full` | 設定されたホストパス上でサンドボックスの外で実行し、承認をスキップします |
| `/elevated off` | サンドボックス内に閉じ込められた実行へ戻します |

`/elev on|off|ask|full` としても利用できます。

現在のレベルを確認するには、引数なしで `/elevated` を送信します。

## 仕組み

- ### 利用可否を確認する
	昇格は設定で有効にされている必要があり、送信者は許可リストに含まれている必要があります。
	json5
	```
	{
	  tools: {
	    elevated: {
	      enabled: true,
	      allowFrom: {
	        discord: ["user-id-123"],
	        whatsapp: ["+15555550123"],
	      },
	    },
	  },
	}
	```
- ### レベルを設定する
	セッションのデフォルトを設定するには、ディレクティブのみのメッセージを送信します。
	Code
	```
	/elevated full
	```
	またはインラインで使用します（そのメッセージにのみ適用されます）。
	Code
	```
	/elevated on run the deployment script
	```
- ### コマンドをサンドボックスの外で実行する
	昇格が有効な場合、 `exec` 呼び出しはサンドボックスの外へ出ます。有効なホストは デフォルトでは `gateway` 、設定済みまたはセッションの exec ターゲットが `node` の場合は `node` です。 `full` モードでは、exec 承認はスキップされます。 `on` / `ask` モードでは、 設定された承認ルールが引き続き適用されます。

## 解決順序

1. メッセージ上の **インラインディレクティブ** （そのメッセージにのみ適用）
2. **セッションオーバーライド** （ディレクティブのみのメッセージを送信して設定）
3. **グローバルデフォルト** （設定内の `agents.defaults.elevatedDefault` ）

## 利用可否と許可リスト

- **グローバルゲート**: `tools.elevated.enabled` （ `true` である必要があります）
- **送信者許可リスト**: チャンネルごとのリストを持つ `tools.elevated.allowFrom`
- **エージェントごとのゲート**: `agents.list[].tools.elevated.enabled` （さらに制限することのみ可能）
- **エージェントごとの許可リスト**: `agents.list[].tools.elevated.allowFrom` （送信者はグローバルとエージェントごとの両方に一致する必要があります）
- **Discord フォールバック**: `tools.elevated.allowFrom.discord` が省略されている場合、 `channels.discord.allowFrom` がフォールバックとして使用されます
- **すべてのゲートに合格する必要があります** 。そうでない場合、昇格は利用不可として扱われます

許可リストエントリの形式:

| プレフィックス | 一致対象 |
| --- | --- |
| （なし） | 送信者 ID、E.164、または From フィールド |
| `name:` | 送信者の表示名 |
| `username:` | 送信者のユーザー名 |
| `tag:` | 送信者タグ |
| `id:`, `from:`, `e164:` | 明示的な ID 指定 |

## 昇格が制御しないもの

- **ツールポリシー**: `exec` がツールポリシーによって拒否されている場合、昇格で上書きすることはできません。
- **ホスト選択ポリシー**: 昇格は `auto` を自由なクロスホスト上書きに変えるものではありません。設定済みまたはセッションの exec ターゲットルールを使用し、ターゲットがすでに `node` の場合にのみ `node` を選択します。
- **`/exec` とは別**: `/exec` ディレクティブは、承認された送信者向けにセッションごとの exec デフォルトを調整するもので、昇格モードを必要としません。

> [!note] Note
> **Note**
> 
> bash チャットコマンド（`!` プレフィックス、 `/bash` エイリアス）は別のゲートであり、独自の `tools.bash.enabled` フラグに加えて `tools.elevated` が有効である必要があります。昇格を無効にすると、`!` シェルコマンドもロックアウトされます。

## 関連[**Exec ツール**

エージェントからのシェルコマンド実行。

](https://docs.openclaw.ai/ja-JP/tools/exec)

[

**Exec 承認**

`exec` の承認および許可リストシステム。

](https://docs.openclaw.ai/ja-JP/tools/exec-approvals)[

**サンドボックス化**

Gateway レベルのサンドボックス設定。

](https://docs.openclaw.ai/ja-JP/gateway/sandboxing)[

**サンドボックス vs ツールポリシー vs 昇格**

ツール呼び出し中に 3 つのゲートがどのように合成されるか。

](https://docs.openclaw.ai/ja-JP/gateway/sandbox-vs-tool-policy-vs-elevated)