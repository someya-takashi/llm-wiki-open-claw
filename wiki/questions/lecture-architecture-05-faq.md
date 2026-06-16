---
type: question
asked: 2026-06-15
question: "OpenClaw について社内でよく出る誤解・不安に、アーキテクチャの知識で答えると？"
series: lecture-architecture
order: 5
sources_used:
  - "[[concepts/exec]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/security]]"
  - "[[concepts/secrets]]"
  - "[[concepts/pairing]]"
  - "[[concepts/groups]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/local-models]]"
  - "[[concepts/remote-access]]"
---

# 05. よくある誤解 Q&A

> 前へ ← [[lecture-architecture-04-why-not-dangerous]] ｜ 目次 → [[lecture-architecture-00-index]]

最後に、社内でよく出る不安に、これまでの章（アーキテクチャと安全の 4 柱）の知識で答えます。各回答は「**構造でどう抑えられているか**」を示します。

---

### Q1. AI が暴走して本番環境を消したりしない？

**A. 「コマンドを実行できる」状態にするには複数のゲートを自分で開ける必要があり、既定では何重にも止まっています。**

シェル実行（[[concepts/exec]]）は、ツールポリシー（そもそも見えるか）→ サンドボックス（どこで動くか）→ 昇格（隔離の外に出るか）→ Exec 承認（許可リスト＋ユーザー確認）という**多重ゲート**を通ります（[[concepts/sandboxing]]）。設定で**実行のたびに人間へ確認**を求められますし（`exec.ask`）、隔離環境（既定 Docker）に閉じ込めれば本番ホストには届きにくい。全自動即実行（YOLO）は**自分で明示的に選ぶ例外**であって既定ではありません。

---

### Q2. 会話やデータが外部に漏れない？

**A. セルフホストなので、データは自分のインフラに留まります。秘密値も平文を避ける仕組みがあります。**

OpenClaw は**あなたの PC / サーバー**で動く（クラウド SaaS 丸投げではない）。会話や認証情報が知らないベンダーのサーバーに溜まりません。API キーなどの秘密値は **SecretRef（[[concepts/secrets]]）**で環境変数 / ファイル / 外部コマンド経由に外部化し、設定ファイルに平文で書かずに済みます。「1 Gateway = 1 信頼境界」（[[concepts/security]]）の原則で、共有が必要なら境界自体を分けます。

---

### Q3. 誰でも AI に話しかけられてしまわない？

**A. 接続認証・端末ペアリングという関門があり、登録されていない相手や端末は入れません。**

ゲートウェイへの接続には接続認証（`gateway.auth.mode`）がかかり、端末は**ペアリング（[[concepts/pairing]]）**で「信頼済み」に登録された分だけが手足として機能します。公開運用で認証なし（`none`）にしない、というのが基本です。「誰でも」ではなく「**登録した人・端末だけ**」が原則。

---

### Q4. 結局クラウドの AI に丸投げで、ベンダーロックインでは？

**A. 橋渡しはしますが、丸投げではありません。モデルは差し替え可能で、ローカルモデルも使えます。**

OpenClaw 自体はあなたのインフラで動く中央ゲートウェイで、**頭脳（モデル）はスポークとして差し替え可能**です（[[concepts/model-providers]]）。Anthropic・OpenAI などのクラウドモデルだけでなく、**手元で動かすローカルモデル（[[concepts/local-models]]）**も選べます。どのモデルを使うかはあなたが決められ、特定ベンダーに固定されません。

> 補足：人格（どう振る舞うか）は agent 側に定義され、頭脳を替えても保たれます（→ [[lecture-architecture-03-message-flow]]）。

---

### Q5. リモートから使うと危なくない？

**A. リモート接続にも同じ認証が効きます。公開の仕方には推奨パターンがあります。**

接続認証はローカル / リモートを問わず**全接続に適用**されます。外から使う場合も、リバースプロキシに認証を委任する `trusted-proxy` などの方法が用意されています（[[concepts/remote-access]] / [[concepts/authentication]]）。「インターネットに認証なしで晒す」のがダメなのであって、リモート利用そのものが危険なわけではありません。

---

### Q6. グループチャットに入れたら事故りそう？

**A. グループは構造上リスクが上がる経路なので、メンション必須や高リスク機能の制限で設計的に塞ぎます。**

オープンなグループ（[[concepts/groups]]）は、不特定の発言が AI に届く＝**プロンプトインジェクション**の経路になり得ます。そのため、グループ／チャネル由来のセッションは（main ではないため）**サンドボックス対象になりやすい**設計で、さらに「メンション必須にする」「グループで昇格 exec を許さない」といった強化が推奨されます（[[concepts/security]]）。`openclaw security audit` が「オープングループ＋昇格」のような危険な組み合わせを警告します。

---

## 講義のまとめ

- **形が見えれば怖くない**：OpenClaw は自分のインフラに置く AI の**中央ゲートウェイ**（ハブ＆スポーク）。
- **AI は配線の上を流れるだけ**：宛先はルーティングが決め、AI は自由に送り先を選べない（[[lecture-architecture-03-message-flow]]）。
- **安全は構造で担保**：知能より前にアクセス制御。認証3層・ツール実行の多重ゲート・脅威モデルの 4 柱（[[lecture-architecture-04-why-not-dangerous]]）。
- だから「**よくわからないが危険**」は、「**仕組みが分かれば、危険は構造で管理されている**」に置き換わります。

深掘りは各章末リンクの概念ページへ。質問は歓迎です。

## 出典

- [[concepts/exec]] / [[concepts/sandboxing]] / [[concepts/security]] / [[concepts/secrets]]
- [[concepts/pairing]] / [[concepts/groups]]
- [[concepts/model-providers]] / [[concepts/local-models]] / [[concepts/remote-access]]

> 目次へ戻る → [[lecture-architecture-00-index]]
