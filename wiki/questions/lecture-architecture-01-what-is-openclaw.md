---
type: question
asked: 2026-06-15
question: "OpenClaw とは何で、何を解決するソフトウェアなのか？ なぜ『セルフホスト』が中核なのか？"
series: lecture-architecture
order: 1
sources_used:
  - "[[overview]]"
  - "[[concepts/architecture]]"
  - "[[components/gateway]]"
  - "[[concepts/channel-routing]]"
  - "[[concepts/model-providers]]"
---

# 01. OpenClaw とは何か

> 前へ ← [[lecture-architecture-00-index]] ｜ 次へ → [[lecture-architecture-02-gateway-hub-spoke]]

## 30 秒でいうと

> **OpenClaw は、自分のインフラ（PC・サーバー）で動かす、AI アシスタント用のセルフホスト型ゲートウェイ。**
> WhatsApp・Telegram・Slack・Discord・Signal・iMessage・WebChat などの**チャットアプリ**と、Anthropic（Claude）・OpenAI などの**AI モデル**を、**1 つの窓口（ゲートウェイ）で橋渡し**する。

「セルフホスト」とは、**他社のクラウド SaaS に丸投げするのではなく、自分が管理する場所でソフトを動かす**こと。これにより、やりとりするデータの所有権を自分の手元に保てます。

## たとえ：自宅に雇う「執事」

OpenClaw は、**あなたの家（インフラ）に住み込みで雇う執事**だと考えると分かりやすいです。

- 執事は、玄関・電話・郵便（＝ Slack や WhatsApp などの**チャネル**）からの来客・連絡をすべて**1 人で受ける**。
- 用件に応じて、必要なら外部の専門家（＝ Claude や OpenAI などの **AI モデル＝頭脳**）に相談して、返事をまとめる。
- 執事はあなたの家に居るので、**家の鍵や情報を外の見知らぬ会社に預ける必要がない**。雇い主（あなた）が「誰を家に入れるか」「何をしてよいか」を決められる。

この「家に住み込み」「1 人の窓口がすべてを受ける」というイメージが、次章の **ハブ＆スポーク**（中央集約）の核になります。

## 何を解決するのか

AI アシスタントを実用しようとすると、こんな課題にぶつかります。OpenClaw はそれぞれに答えを持っています。

| 課題 | ありがちな状況 | OpenClaw の答え |
|---|---|---|
| **チャネルがバラバラ** | Slack 用 bot、LINE 用 bot…と個別に作り込む | 複数チャネルを 1 つのゲートウェイに集約（[[concepts/channel-routing]]） |
| **モデルに縛られる** | 特定の AI ベンダーに固定される | 複数プロバイダーを差し替え可能（[[concepts/model-providers]]） |
| **データを預けたくない** | クラウド SaaS に会話・認証情報が残る | セルフホストでデータ所有権を保持 |
| **窓口が分散して管理不能** | 誰が・何を・どこで実行したか追えない | 単一ゲートウェイに制御点を集約（[[components/gateway]]） |

最後の「**制御点を 1 つに集約する**」が、講義後半の安全性の話に直結します。窓口が 1 つなら、「誰が話せるか」「何をしてよいか」の関門もそこ 1 か所で管理できるからです。

## 3 つの登場人物（次章への前振り）

OpenClaw を理解するうえで、まずこの 3 者を押さえれば十分です。

1. **ゲートウェイ（Gateway）** — すべての接続を所有する、常駐の中央プロセス（執事本体）。[[components/gateway]]
2. **チャネル（Channel）** — Slack や WhatsApp など、人間が話しかけてくる入口。[[concepts/channel-routing]]
3. **モデルプロバイダー（Model Provider）** — Claude / OpenAI など、考える頭脳を提供する側。[[concepts/model-providers]]

これらが**どう繋がっているか**を次章で図にします。形が見えれば、「よくわからない」はかなり消えます。

## この章のまとめ

- OpenClaw ＝ **自分のインフラで動かす AI アシスタントの中央ゲートウェイ**。クラウド丸投げではない。
- チャットアプリと AI モデルを **1 つの窓口で橋渡し**し、**データ所有権**を保つ。
- 「窓口が 1 つ」という設計が、管理しやすさと安全性の土台になる。

## 出典

- [[overview]] — OpenClaw 全体像
- [[concepts/architecture]] — Gateway 中心アーキテクチャ
- [[components/gateway]] / [[concepts/channel-routing]] / [[concepts/model-providers]]

> 次へ → [[lecture-architecture-02-gateway-hub-spoke]]
