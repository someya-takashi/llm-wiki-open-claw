---
type: translation
of: "[[articles/openclaw-nvidia-skill-security]]"
source_url: https://openclaw.ai/blog/openclaw-nvidia-skill-security
lang: ja
translated: 2026-06-15
---

# OpenClaw、より強力なエージェントスキルセキュリティのため NVIDIA と協業（全文翻訳）

> 原典: `raw/articles/OpenClaw Collaborates with NVIDIA for Stronger Agent Skill Security.md` ・ https://openclaw.ai/blog/openclaw-nvidia-skill-security
> 要約・既存 wiki との接続は [[articles/openclaw-nvidia-skill-security]]
> ℹ️ これは原文（英語）を一文一文忠実に全訳したものです。コードブロック・リンク URL・画像参照は原文のまま保持しています。

---

エージェントスキルファイルは安全でないという評判があり、その評判は当然のものです。

私たちが OpenClaw とともに ClawHub をローンチしたとき、私たちはすぐに、既知のマルウェアを同梱したスキルを公開しようとする者たちに狙われました。私たちは [VirusTotal と提携](https://openclaw.ai/blog/virustotal-partnership)して、それらのスキルにフラグを立て、公開者を自動的に banしました。

従来のマルウェアスキャンは、比較的解決済みの問題です。*エージェント的（agentic）*なリスクを特定することは、そうではありません。あるスキルは、あなたのログを要約すると主張しつつ、それらをあなたのマシンから送り出すスクリプトを同梱していることがあります。善意のスキルが、誤ったフラグで本番環境を消し去る CLI にあなたのエージェントを向けてしまうことがあります。どちらも古典的な意味でのマルウェアではなく、どちらもウイルススキャナーが捕らえるために作られたものではありません。

ですからスキルをインストールする前に、あなたは本当に 3 つのことを知りたいはずです。

1. それが何をすると主張しているか。
2. 同梱されたコードが実際にその主張と一致するか。
3. 何か問題が起きたとき、影響範囲（blast radius）がどう見えるか。

それらの問いにエコシステム規模で答えることは、ClawHub の使命の中核です。OpenClaw を最大限に活用するには、私たちのユーザーは、自分がインストールするスキルやプラグインが徹底的に検証されていると信頼できる必要があります。本日、私たちはその作業をどう行っているか、それを測定したときに何が分かったか、そしてコミュニティの他の人々がそれを土台にできるよう公開データセットを共有します。

## ClawScan パイプライン

信頼を構築する私たちの最初の試みは、[OWASP のエージェント的リスク](https://owasp.org/www-project-agentic-skills-top-10/)を探すよう促された Codex エージェントでした。それは機能し、実際の悪意ある行為者を捕らえました。しかしそれはクローズドソースの取り組みであり、エージェント的リスクの問題は、いかなる単一のレジストリも単独で防御するには新しすぎ、速く動きすぎています。

ですから私たちは今、NVIDIA の[検証済みエージェントスキルイニシアチブ](https://developer.nvidia.com/blog/nvidia-verified-agent-skills-provide-capability-governance-for-ai-agents/)で、その作業をオープンに行いながら協業しています。ClawHub を通って流れるすべてのスキルは、公開される前に、カタログ前の検証ゲートを通過します。

![ClawScan の信頼パイプライン](https://openclaw.ai/blog/openclaw-nvidia-skill-security/clawscan-trust-pipeline.png)

新しいスキルバージョンが公開されると、OpenAI Codex エージェントが、3 つの独立したスキャナーの出力をコンテキストとして受け取ります。私たちの静的解析、VirusTotal、そして NVIDIA SkillSpector です。Evaluate ステップである ClawScan は、来歴（provenance）、メタデータ、モデレーション履歴と並べて 3 つすべてを比較検討し、最終判定とともに Skill Card を生成します。判定は Clean、Suspicious、または Malicious です。

## NVIDIA Skill Cards と SkillSpector の協業

そのセキュリティプロセスの 2 つの部分は新しく、どちらも NVIDIA との協業から生まれたものです。

**NVIDIA Skill Cards** はオープンな信頼アーティファクト仕様であり、現在はすべての公開スキルに同梱されます。各カードは、誰が公開したか、それが何をできるか、ClawScan が何を見つけたか、そしてそれが正確にどこから来たかを伝えます。これらはすべて、公開者の自己申告から取られるのではなく、ClawHub によって検証されます。スキル詳細ページのタブで読むか、ターミナルから `openclaw skills verify <slug> --card` で読めます。

![ClawHub の Skill Card](https://openclaw.ai/blog/openclaw-nvidia-skill-security/skill-card.png)

**NVIDIA SkillSpector** は新しいエージェントスキルスキャナーです。静的チェックと AI 支援のセマンティック解析を組み合わせ、マルウェアスキャナーが見逃すリスクにフラグを立てます。隠された指示、危険なコードパス、過度に広い権限、依存関係の問題、そしてスキルが宣言した目的と実際の振る舞いの不一致です。ClawHub では、SkillSpector の発見は**助言（advisory）**として表示されます。それらは自動的にスキルをブロックしません。ClawScan は、判定に至る前に、それらを他のすべてと並べて比較検討します。

![ClawHub の SkillSpector の発見](https://openclaw.ai/blog/openclaw-nvidia-skill-security/skill-spector.png)

## 初期の発見

私たちの想定は、これら 3 つのスキャナーの結果がほとんど重なるだろうというものでした。ところが、それらはほとんどまったく重なりません。

| スキャナーの組 | 両方が陽性 | Jaccard 一致度 |
| --- | --- | --- |
| VirusTotal と SkillSpector | 3,286 | 0.094 |
| 静的解析と SkillSpector | 3,511 | 0.104 |
| 静的解析と VirusTotal | 586 | 0.065 |

どの組も、合算した陽性の 10.4% を超えて一致しません。468 スキル、すなわち 0.69% だけが、3 つのスキャナーすべてで同時にフラグされます。陽性の発見の 81.9% は、単一のスキャナーのみから来ています。

私たちは、これらの発見が個々のスキャナーのいずれかの問題を示しているとは考えていません。むしろ、各スキャナーは異なるリスク表面を持っています。VirusTotal はマルウェアの評判を見、静的解析は危険なコードパターンを見、SkillSpector はエージェント的リスクを見ます。全 67,453 行のうち、SkillSpector は 32,856 行、すなわち 48.71% で陽性であり、対して VirusTotal は 5,225 行、すなわち 7.75%、静的解析は 4,434 行、すなわち 6.57% です。ClawScan が suspicious と判定した 25,504 行の中では、SkillSpector は 19,209 行、すなわち 75.3% で陽性です。206 の malicious 行の中では、パターンが逆転します。VirusTotal は 150 行、すなわち 72.8% で陽性であり、一方 SkillSpector は 14 行、すなわち 6.8% です。

これらの不一致は、ClawScan のような LLM-as-judge の必要性を浮き彫りにします。広いリスク表面を持つスキルと、真に悪意あるスキルとを区別することは、新規の課題です。

一例として、私たちが特定したあるスキルは SkillSpector から 173 件の発見を受けましたが、ClawScan は依然としてそれを malicious ではなく suspicious とラベル付けしました。

このスキャン結果のコーパスを私たち自身のものに留めておくのではなく、私たちはそれをより広いセキュリティコミュニティのためにオープンソース化して、改善を手伝ってもらえることを嬉しく思います。

## セキュリティスキャンシグナルのデータセットをオープンソース化

ClawHub のスキルセキュリティへのコミットメントは、私たち自身のレジストリで終わりません。私たちが知っていることを共有すれば、コミュニティ全体がより安全になります。

最も人気のあるスキルレジストリの 1 つとして、私たちは現在、毎日数千の公開イベントに対してフルの ClawScan スイートを走らせており、その過程で OpenAI GPT-5.5 を用いて数百万の LLM トークンを消費しています。v1 データセットは 67,453 件の最新の公開スキルバージョンをカバーします。これ自体が、セキュリティ研究コミュニティにとって計り知れないほど貴重な膨大なシグナルを生成していますが、これまでは ClawHub の内部に閉じ込められていました。

**本日、私たちは ClawHub のすべてのセキュリティスキャン結果の公開データセットを Hugging Face で公開します: [OpenClaw/clawhub-security-signals](https://huggingface.co/datasets/OpenClaw/clawhub-security-signals)。** このプロジェクトへの貢献について、NVIDIA の Jacob Tomlinson、Agustin Rivera、Michael Appel に特に感謝します。

私たちの願いは、これがより広い研究コミュニティに力を与え、スキルセキュリティツールの最先端を改善する私たちの取り組みに加わってもらうことです。この作業は、エージェントスキルエコシステム全体を保護し、より広い AI エコシステムを支えるという私たちの使命の中核です。

上げ潮はすべての爪（claw）を持ち上げます。🦞

> [!note] Note
> 完全な方法論とスキャナー間の不一致の分析については、コンパニオン論文 [ClawHub Security Signals: When VirusTotal, Static Analysis, and SkillSpector Disagree](https://openclaw.ai/publications/clawhub-security-signals.pdf) を読んでください。
