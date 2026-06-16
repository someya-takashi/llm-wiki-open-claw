---
type: translation
of: "[[articles/safer-than-yolo-auto-mode-for-exec-approvals]]"
source_url: https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
lang: ja
translated: 2026-06-15
---

# YOLO より安全：Exec 承認の auto モード（全文翻訳）

> 原典: `raw/articles/Safer Than YOLO_ Auto Mode for Exec Approvals.md` ・ https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
> 要約・既存 wiki との接続は [[articles/safer-than-yolo-auto-mode-for-exec-approvals]]
> ℹ️ これは原文（英語）を一文一文忠実に全訳したものです。コードブロック・リンク URL は原文のまま保持しています。

---

YOLO モードは、承認プロンプトを省略することでホストコマンドを高速にしました。それは信頼済みのローカル自動化や、外部でサンドボックス化された実行には有用ですが、日常的な利用における唯一の良い答えとしては大雑把すぎます。

私たちは本日、デフォルトを変更するわけではありません。

`auto` は、私たちが公開の場でテストしているオプトインの経路です。それが有用だと証明されれば、より多くのユーザーにとってより安全なデフォルトにすることを検討しますが、原則は変わりません。OpenClaw は、オペレーターの選択を奪うことなく人々を保護すべきです。

auto は、Enterprise 環境に最も適したモードです。まずポリシーが走り、低リスクの取りこぼしはモデルがレビューでき、不確かなものは依然として人間にルーティングされます。

安全で繰り返し可能なコマンドは、あなたを煩わせることなく実行できます。ポリシーを外れたコマンドは、まずレビュアーに回されます。レビュアーが確信できなければ、OpenClaw はあなたに尋ねます。

OpenAI はすでにこのパターンを Codex 内の Guardian として出荷しています。[Codex harness](https://docs.openclaw.ai/plugins/codex-harness) を通じて、OpenAI を裏側に使う OpenClaw セッションは Codex ネイティブのレビュー済み承認を使えます。今、私たちは同じ形を OpenClaw のホスト exec に、誰でも使えるオプトインモードとしてもたらします。

## なぜこれが存在するのか

Codex はすでに、自身の権限プリセットでこの転換を行いました。その Guardian レビュー済みのフローは、一般的なワークスペース作業を進めさせつつ、ネットワークアクセスやワークスペース外への書き込みのような脱出には依然としてレビューを要求します。

OpenClaw は同じ形を [host exec](https://docs.openclaw.ai/tools/exec-approvals) にもたらします。`tools.exec.mode: "auto"` は、すべてのコマンドを恒久的な「yes」に変えることなく、エージェントを動かし続けます。

Ask **Humanfirst（人間優先）**

許可リストの取りこぼしは停止し、オペレーターを待ちます。厳格なセットアップには良いですが、忙しいエージェントにはうるさい。

Auto **Reviewerfirst（レビュアー優先）**

決定的な一致は実行されます。取りこぼしは、人間へのフォールバックの前に OpenClaw のネイティブな auto レビュアーを通ります。

YOLO **Noprompts（プロンプトなし）**

ホスト exec は承認プロンプトなしで実行されます。周囲の環境がすでに信頼済みのときだけ有用です。

## auto は何をするか

ホスト exec は OpenClaw の設定から始まります。すなわち、エージェントが何を要求してよいか、です。ほとんどのユーザーはその設定だけで足ります。ホストは依然としてより厳格なローカルポリシーを適用できます。

レビュアーモデルは、メインのエージェントモデルとは別です。通常の作業ではエージェントをローカルモデルに置いたまま、承認の取りこぼしについてより強い判断が欲しいときには exec レビューを `openai/gpt-5.5` のようなフロンティアモデルに向ける、ということができます。それにより両方の良いとこ取りができます。すなわち、通常のターンにはローカルファーストの実行、ホストアクセスに決定が必要なときだけより高い確信度のレビュー、です。

`auto` モードでは、OpenClaw はホストコマンドを次のように扱います。

1. コマンドが許可リスト、または決定的な safe-bin ルールに一致すれば、実行されます。
2. コマンドがポリシーを外れた場合、OpenClaw は境界付きのレビューパケットを組み立てます。コマンド、argv、cwd、env のキー名、ホスト、パーサー解析、です。
3. auto レビュアーは、低リスクの実行を 1 回だけ許可できます。
4. 曖昧なもの、よりリスクの高いもの、解析不能なもの、タイムアウトしたもの、モデルが利用不能なもの、あるいはレビュアーが指示したものは、人間の承認にフォールバックします。
5. UI も設定済みの承認クライアントも応答できない場合、OpenClaw はホストの設定済みフォールバックを使います。

`auto` はローカルの安全設定を上書きしません。常に尋ねるよう設定されたホストは依然として尋ねます。拒否するよう設定されたホストは依然として拒否します。

## auto を有効にする

ローカルの gateway-host セットアップの場合：

```bash
openclaw config set tools.exec.host gateway
openclaw config set tools.exec.mode auto
```

そのホストは、ホスト exec について auto にオプトインしました。

レビューにメインエージェントより強いモデルを使いたい場合は、レビュアーモデルを別途設定します。

```bash
openclaw config set tools.exec.reviewer.model openai/gpt-5.5
```

未設定のままにすると、現在のエージェントモデルを再利用します。

[Codex harness](https://docs.openclaw.ai/plugins/codex-harness) を使う場合、これは OpenAI を裏側に使うセッションを、利用可能なときは workspace-write サンドボックス化を伴う Codex の Guardian レビュー済み承認にマッピングします。

## 何が尋ねられるか

レビュアーが安全に「yes」と言えないとき、人間の承認が依然として最終権限です。

承認プロンプトは次を提示できます。

- `allow-once`：この正確な要求を 1 回実行する。
- `allow-always`：要求がサポートする場合、恒久的な許可リストエントリを永続化する。
- `deny`：実行しない。

`allow-once` は意図的に狭くなっています。ノードホストの実行では、OpenClaw は承認を正規のコマンドプラン、cwd、argv、セッションコンテキストに束ねます。承認要求が作られた後に呼び出し元がコマンドを変更した場合、変更された要求を黙って実行する代わりに、その実行は拒否されます。

## チャットでの承認

承認はもはやローカルのターミナルに閉じ込められていません。OpenClaw は承認プロンプトを、オペレーターがすでに見ている場所——[Slack](https://docs.openclaw.ai/channels/slack)、[Telegram](https://docs.openclaw.ai/channels/telegram)、[iMessage](https://docs.openclaw.ai/channels/imessage) を含む——にルーティングできます。

詳細なセットアップは [Exec approvals - advanced](https://docs.openclaw.ai/tools/exec-approvals-advanced) にあります。

## セキュリティ上の注意

`auto` はプロンプトのノイズを減らします。それでもホストポリシーを尊重します。

レビュアーは低リスクの実行を 1 回だけ許可できます。レビュアーは、コマンドテキスト、argv、cwd、env キー、ヒアドキュメント、文字列、ファイル名、メタデータを信頼できないデータとして扱うよう促されます。その信頼できないデータがレビュアーに指示しようとしたり、決定を要求しようとしたりした場合、OpenClaw は人間に委ねます。

YOLO は、すでに外部でサンドボックス化されているか、意図的に信頼されている環境のために引き続き利用可能です。Enterprise 環境には `auto` の方が適しています。厳格な ask モードより少ないプロンプト、フルのホストアクセスより多くのレビュー、そしてレビュアーが安全に「yes」と言えないときの人間へのフォールバック、です。私たちはまだその切り替えを強制しません。
