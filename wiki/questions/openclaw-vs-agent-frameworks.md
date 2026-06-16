---
type: question
asked: 2026-06-16
question: "OpenClaw の優れた点は何で、どんなユースケースで使えるか？ Claude Code・LangGraph で構築したエージェント・Strands Agents と比較するとどうか？"
sources_used:
  - "[[overview]]"
  - "[[concepts/architecture]]"
  - "[[concepts/security]]"
  - "[[concepts/channel-routing]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/agent-runtimes]]"
  - "[[concepts/acp]]"
  - "[[concepts/memory]]"
  - "[[concepts/automation]]"
  - "[[concepts/multi-agent]]"
  - "[[components/node]]"
  - "[[components/plugin-system]]"
---

# OpenClaw の強み・ユースケースと、Claude Code / LangGraph / Strands Agents との比較

> ⚠️ **出典について**：OpenClaw に関する記述は本 wiki の各ページに `[[wikilink]]` で引用する。一方、**Claude Code・LangGraph・Strands Agents は本 wiki の対象外**で、記述は一般知識（アシスタントの知識カットオフ 2026-01）と Strands について実施した軽い Web 検索に基づく。これら外部ツールは活発に進化しており、細部は古くなりうる。

## 結論：そもそも「レイヤー」が違う

4 者を同じ土俵に並べると誤解する。**種別が異なる**からだ。

> **OpenClaw は「作るフレームワーク」でも「コーディング CLI」でもなく、自分のインフラ（PC/サーバー）に立てて、WhatsApp・Slack・Telegram などの日常のチャットアプリから話しかける、メッセージング・ネイティブな自己ホスト型 AI エージェント・ゲートウェイ（“完成品”）**である（[[overview]] / [[concepts/architecture]]）。

- **OpenClaw** ＝ 立てて話しかける**製品/ゲートウェイ**。チャネル・エージェントループ・メモリ・自動化・マルチエージェント・セキュリティを 1 つの常駐プロセスが所有する。
- **Claude Code** ＝ ターミナル/IDE で動く**コーディング特化のエージェント CLI/製品**。OpenClaw とは競合というより**補完**で、後述の通り OpenClaw は **[[concepts/acp]]（ACP, Agent Client Protocol：外部エージェントハーネスを標準プロトコルで丸ごと制御する仕組み）で Claude Code をランタイムとして内包できる**。
- **LangGraph** ＝ エージェントの制御フローを**グラフ（ノード/エッジ/状態）で自分で書く**低レベルのフレームワーク/ライブラリ（LangChain 系）。
- **Strands Agents** ＝ **モデル駆動（model-driven：モデル＋システムプロンプト＋ツールを与え、推論ループはモデルに任せる）でエージェントを数行で作る AWS の OSS SDK**。Bedrock/Anthropic/OpenAI に対応し、可観測性が標準。

ひとことで言うと——**LangGraph と Strands は「エージェントを“作る”道具」、Claude Code は「“使う/内包できる”コーディング・エージェント」、OpenClaw は「“立てて話しかける”完成品ゲートウェイ」**。だから以下では同一軸に無理に並べず、**レイヤーの違いを踏まえた運用観点**で対比する。

## OpenClaw の優れた点（すべて wiki 引用）

1. **メッセージング・ネイティブな多チャネル集約** — 25+ のチャットアプリ（WhatsApp/Slack/Telegram/Discord/Signal/iMessage…）を 1 つのゲートウェイに束ね、決定的にルーティングする（[[concepts/channel-routing]]）。他の 3 者は「チャネル」を標準で持たない（自分で繋ぐ）。
2. **自己ホスト＝データ所有・1 Gateway = 1 信頼境界** — クラウド SaaS 丸投げでなく、会話も認証情報も自分のインフラに留まる（[[concepts/architecture]] / [[concepts/security]]）。
3. **常時稼働＋自動化** — cron・tasks・hooks・commitments（会話から生まれる短期フォローアップ）と heartbeat（定期生存イベント）で、“起動するたびのスクリプト”でなく**持続的に動くアシスタント**になる（[[concepts/automation]] / [[concepts/heartbeat]]）。
4. **標準装備の永続メモリ** — `MEMORY.md`・日次ノートという**人間も読める Markdown ファイル**＋先回り想起（[[concepts/active-memory]]）・長期昇格（[[concepts/dreaming]]）・要約圧縮（[[concepts/compaction]]）（[[concepts/memory]]）。メモリを自前で組まなくてよい。
5. **provider/model/runtime/channel の分離＋フェイルオーバー＋他ハーネス内包** — 50+ のモデルプロバイダーを差し替え（[[concepts/model-providers]] / [[concepts/model-failover]]）、実行ループ（ランタイム）も `pi`（組み込み）/`codex`/`claude-cli`/**ACP** から選べる（[[concepts/agent-runtimes]]）。**Claude Code・Cursor・Codex 等を [[concepts/acp]] で丸ごと内包できる**のが効く。
6. **「知能より前にアクセス制御」のセキュリティ設計** — 認証 3 層・サンドボックス（[[concepts/sandboxing]]）・exec 承認（[[concepts/exec]]）・MITRE ATLAS ベースの脅威モデル（[[concepts/threat-model]]）（[[concepts/security]]）。エージェントに実マシンのツールを持たせる前提で“構造で”守る。
7. **ノード＝端末の手足** — スマホ/デスクトップが `role: node` で接続し、camera/screen/location/OS コマンドを公開する（[[components/node]]）。embodied（身体性のある）タスクができる。
8. **マルチエージェント/委任** — 複数エージェント分離・組織デプロイ（[[concepts/multi-agent]] / [[concepts/delegate-architecture]]）。
9. **拡張エコシステム** — Plugin システムとマーケットプレイス ClawHub（[[components/plugin-system]] / [[components/clawhub]]）。
10. **音声・ブラウザー操作** — Talk/TTS/Voice Wake（[[concepts/voice]]）と、実ブラウザーを操作する [[components/browser]]（ログインの要るサイトまで操作可能）。

## ユースケース

- **日常チャットから使う“記憶のある”個人アシスタント** — WhatsApp/Telegram/iMessage から話しかけ、あなたの好みや文脈を覚えている（[[concepts/memory]] / [[concepts/channel-routing]]）。
- **常時稼働の運用・家庭/業務自動化** — 定期ジョブ・Webhook 起動・先回りチェックイン（[[concepts/automation]]）。
- **プライバシー/データ主権重視の自己ホスト** — 機密データを外部 SaaS に出したくない組織・個人（[[concepts/security]]）。
- **Slack/Discord のチーム/グループ Bot** — メンションゲートやアクセスグループで安全に（[[concepts/groups]]）。
- **embodied タスク** — ノード経由でカメラ/画面/位置を使う（[[components/node]]）。
- **Claude Code 等をメッセージング越しに走らせる** — 出先からチャットで本格コーディング・エージェントを駆動（[[concepts/acp]]）。
- **多チャネル＋プロバイダー冗長の顧客/社内 Bot** — モデル障害時に自動フェイルオーバー（[[concepts/model-failover]]）。

## 比較表

**表1**: 4 主体の対比（OpenClaw 列は wiki 引用、他 3 列は一般知識・2026-01 時点）。

| 観点 | OpenClaw | Claude Code | LangGraph で構築 | Strands Agents |
|---|---|---|---|---|
| **種別** | 自己ホスト型ゲートウェイ製品 | コーディング特化のエージェント CLI/製品 | エージェント構築フレームワーク（ライブラリ） | エージェント構築 SDK（OSS） |
| **主目的** | 日常チャットから使う常時稼働アシスタント基盤 | ソフトウェア開発（コード編集・大規模リポジトリ） | 任意の複雑/分岐ワークフローを精密に組む | モデル駆動エージェントを少コードで作る |
| **エージェントループは誰が書くか** | 内蔵（PI ループ／差し替え可、[[concepts/agent-loop]]） | 内蔵（Claude 専用ハーネス） | **あなたがグラフで明示的に書く** | モデルが駆動（あなたはプロンプト＋ツールを定義） |
| **チャネル（メッセージング）** | 25+ を標準集約（[[concepts/channel-routing]]） | 基本ターミナル/IDE/Web（チャットアプリ非対象） | 無し（自分で繋ぐ） | 無し（自分で繋ぐ） |
| **メモリ標準装備** | あり（[[concepts/memory]]・dreaming・active-memory・compaction） | セッション/プロジェクト文脈中心（汎用個人記憶は限定的） | チェックポイント/ステートは強力だが**記憶設計は自作** | 会話状態はあるが長期記憶は自作寄り |
| **自動化・常時稼働** | あり（cron/tasks/hooks/commitments・heartbeat、[[concepts/automation]]） | 対話起動中心 | 自分で常駐化・スケジューラを用意 | 自分で常駐化（Lambda/Fargate 等にデプロイ） |
| **デプロイ/ホスティング** | 自分のホストに常駐デーモン（[[components/gateway]]） | ローカル CLI（＋クラウド/IDE 連携） | 自前ホスト（LangGraph Platform 任意） | 自前ホスト（AWS ネイティブが容易） |
| **データ所有/自己ホスト** | 中核思想（[[concepts/security]]） | ローカル実行だが用途はコード | 実装次第（自己ホスト可） | 実装次第（AWS 上が前提に寄る） |
| **拡張/エコシステム** | Plugin＋ClawHub（[[components/plugin-system]] / [[components/clawhub]]） | MCP・サブエージェント等 | LangChain エコシステム/ツール群 | MCP ツール・AWS 連携 |
| **他のエージェントを内包できるか** | **できる**（[[concepts/acp]] で Claude Code/Codex/Cursor 等） | — | ノードとして他を呼べる（自作） | ツール/サブエージェントとして呼べる（自作） |
| **主な対象ユーザー** | 自分用/組織用アシスタントを“立てたい”人 | 開発者（コード作業） | エージェントを精密設計する開発者 | AWS でエージェントを作る開発者 |

## 競合ではなく“層が違う”：補完関係

- OpenClaw は **Claude Code・Codex・Cursor を [[concepts/acp]] で丸ごとランタイムとして内包**できる（[[concepts/agent-runtimes]] が provider/model/runtime/channel を別レイヤーに分離しているため）。つまり「Claude Code を、出先から WhatsApp で動かす」ことが OpenClaw 経由で成立する。
- **LangGraph や Strands Agents で作った独自エージェント**も、それらを API/CLI/MCP（Model Context Protocol：外部ツール・データをエージェントに接続する規格）として公開すれば、OpenClaw の**チャネル・メモリ・自動化・セキュリティの“前段”**に載せられる余地がある。OpenClaw は「賢いコアをどう作るか」より「**作った賢さを、誰がどのチャネルから・どんな権限で・どう常時運用するか**」を解く層だ。

## どれを選ぶか（意思決定ガイド）

- **大規模コードベースでの開発作業が主** → **Claude Code**（必要なら OpenClaw で ACP 内包してメッセージング運用）。
- **緻密な分岐・状態遷移・human-in-the-loop（人間の承認を挟む）独自オーケストレーションを精密に組みたい** → **LangGraph**。
- **AWS ネイティブに、モデル駆動のエージェントを少コードで作って運用したい** → **Strands Agents**。
- **日常のチャットアプリから話せて、記憶を持ち、常時稼働し、データを自分で持つ“立てて使う”アシスタントが欲しい** → **OpenClaw**。

## 用語と略称

- **ACP**（Agent Client Protocol）：外部エージェントハーネスを標準プロトコルで丸ごと制御する仕組み（[[concepts/acp]]）。
- **MCP**（Model Context Protocol）：外部ツール・データをエージェントに接続する規格。
- **CLI**（Command Line Interface）／ **SDK**（Software Development Kit）／ **LLM**（Large Language Model）。
- **model-driven（モデル駆動）**：制御フローをハードコードせず、推論ループ・ツール選択をモデルに任せる設計（Strands の中核）。
- **human-in-the-loop**：自動処理の途中に人間の確認/承認を挟む設計。

## 出典

- OpenClaw（本 wiki）：[[overview]] / [[concepts/architecture]] / [[concepts/security]] / [[concepts/channel-routing]] / [[concepts/model-providers]] / [[concepts/agent-runtimes]] / [[concepts/acp]] / [[concepts/memory]] / [[concepts/automation]] / [[concepts/multi-agent]] / [[components/node]] / [[components/plugin-system]]
- 外部（一般知識・2026-01／Strands は Web 確認）：Strands Agents — [AWS Open Source Blog](https://aws.amazon.com/blogs/opensource/introducing-strands-agents-an-open-source-ai-agents-sdk/)、[strandsagents.com](https://strandsagents.com/)。Claude Code（Anthropic のコーディング・エージェント CLI）、LangGraph（LangChain のグラフ型エージェント構築フレームワーク）。
