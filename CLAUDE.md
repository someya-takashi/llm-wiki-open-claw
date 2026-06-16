# OpenClaw LLM Wiki — スキーマ

このリポジトリは Andrej Karpathy が提唱した「LLM Wiki」パターンに基づく、**OpenClaw（あらゆる OS で動作する AI エージェント用のセルフホスト型 Gateway。WhatsApp・Telegram・Slack・Discord・Signal・iMessage・WebChat などのメッセージングサーフェスと AI エージェントを単一のゲートウェイで橋渡しし、ユーザーがデータの所有権を保ったまま複数チャネルから AI アシスタントを使えるようにするソフトウェア）** に関するパーソナル・ナレッジベースです。主な情報源は公式ドキュメント `https://docs.openclaw.ai/ja-JP/` で、Gateway アーキテクチャ、チャネル統合、モデルプロバイダー、エージェントの仕組み（agent loop / memory / context / session）、自動化（cron / hooks / webhook）、ノード（モバイル/デスクトップ端末）、プラグイン、CLI などを扱います。LLM（あなた）が wiki を**読み・書き・更新する側**、ユーザーは**情報源のキュレーションと質問**を担当します。ユーザーは wiki を直接編集することはほぼありません。

このファイルは「あなた（LLM）」のための運用ルール書（スキーマ）です。ingest / query / lint の各オペレーションは skill として切り出してあり（§3）、それらの skill は本ファイルのスキーマ規約に従って実行してください。

---

## 1. ディレクトリ構成

```
OpenClaw/
├── CLAUDE.md                ← このファイル（スキーマ）
├── .claude/skills/          ← オペレーション skill（ingest / query / lint）
├── Clippings/               ← Obsidian Web Clipper の着地点（暫定置き場。ingest 完了後に raw/docs/ の適切なフォルダへ移送する）
├── raw/                     ← 原典（immutable, LLM は読むだけ）
│   ├── docs/                ← 公式ドキュメントの clip（markdown）。docs の URL ツリーをサブフォルダで反映
│   ├── articles/            ← 非公式の記事・ブログ等（**有効**。記事 ingest の入口。ユーザーが直接ここへ配置する。Clippings/ は経由しない）
│   └── assets/              ← 記事内画像・スクリーンショットのローカル保存先
└── wiki/                    ← LLM が完全所有する markdown 群
    ├── index.md             ← カタログ（全ページの一覧）
    ├── log.md               ← 時系列ログ（append-only）
    ├── overview.md          ← OpenClaw 全体の総括ページ（随時更新）
    ├── sources/             ← 原典 1 ページにつき 1 ページの解説（docs ツリーをサブフォルダで反映）
    ├── concepts/            ← 概念・仕組み（抽象レベル）の解説ページ
    ├── components/          ← OpenClaw を構成する具体的サブシステムの解説ページ
    ├── channels/            ← メッセージング統合（Slack / Telegram / WhatsApp 等）の解説ページ
    ├── providers/           ← モデル提供元（Anthropic / OpenAI / Ollama 等）の解説ページ
    ├── articles/            ← 非公式記事の日本語要約・解説（`raw/articles/` に 1:1 対応）
    │   └── translations/    ← 英語記事の全文翻訳（一文一文の忠実訳。英語記事のみ）
    └── questions/           ← query で得た成果物（比較表・分析等）を保存
```

### 命名規約

- ファイル名は `kebab-case.md`（例: `agent-loop.md`、`channel-routing.md`、`3d-secure.md` のような数字始まりも可）。
- **原典の wiki ページ（sources/）は、公式ドキュメントの URL パスをそのままサブフォルダ＋スラグにする**。clip のタイトルではなく **URL 由来**で決める。
  - 例: `https://docs.openclaw.ai/ja-JP/concepts/architecture` → 原典 `raw/docs/concepts/architecture.md`、解説 `wiki/sources/concepts/architecture.md`。
  - 例: `https://docs.openclaw.ai/ja-JP/gateway/protocol` → `raw/docs/gateway/protocol.md`、`wiki/sources/gateway/protocol.md`。
  - こうして **docs のセクション階層（`concepts/` `gateway/` `channels/` `providers/` `cli/` `tools/` `plugins/` 等）が raw と sources の両方でそのまま再現**される。
- **概念ページ（concepts/）は「横断的な概念・仕組み」を単位にする**。スラグは概念の英語名。
  - 例: `architecture`, `agent-loop`, `memory`, `context-engine`, `queue`, `session`, `pairing`, `sandboxing`, `multi-agent`, `channel-routing`, `model-providers`, `presence`, `heartbeat`。
- **OpenClaw を構成する具体的なサブシステム（landmark）は `components/<slug>.md` に専用ページを作ってよい**。
  - 例: `gateway`, `node`, `webchat`, `control-ui`, `cli`, `clawhub`, `plugin-system`。
  - **component ページは必ず、関連する上位概念ページ（concepts）へリンクする**（frontmatter の `concepts` と本文の双方）。逆に概念ページからは、それを体現する代表 component として components/ のページへリンクする。詳細な内部構造・設定・限界は component ページ側に書き、概念ページ側は俯瞰と位置づけに徹する。
  - 専用ページを作るほど確立していない要素は、関連する概念ページ内に記述するに留めてよい。判断に迷ったら概念ページ内記述から始め、言及が増えたら独立させる。
- **メッセージングチャネルは `channels/<slug>.md`、モデルプロバイダーは `providers/<slug>.md` に 1 統合 = 1 ページで作る**。
  - channels の例: `slack`, `telegram`, `whatsapp`, `discord`, `signal`, `imessage`, `matrix`。
  - providers の例: `anthropic`, `openai`, `google`, `ollama`, `openrouter`, `groq`。
  - **channel / provider ページは、関連する構成要素（多くは `[[components/gateway]]`）と概念ページへリンクする**（channels なら `[[concepts/channel-routing]]`、providers なら `[[concepts/model-providers]]` 等。frontmatter の `related` と本文の双方）。
- **非公式記事（ブログ等）は `articles/<slug>.md` に 1 記事 = 1 ページで作る**。スラグは記事の **source URL 末尾**（`https://openclaw.ai/blog/<slug>` → `<slug>`）。`raw/articles/<slug>.md`（原典）↔ `wiki/articles/<slug>.md`（日本語要約・解説）の 1:1 ミラー。英語記事は全文翻訳を `wiki/articles/translations/<slug>.md` に置く。
  - **記事は二次資料**（意見・告知・チュートリアル）として扱う。公式 docs に基づく concept / component の本文を上書きせず、関連する concept / component / source へ**引用リンクで接続**する（記事側から「この記事は [[concepts/exec]] を補足する」、概念側の関連セクションから「ブログで〜が告知 → [[articles/<slug>]]」）。docs より新しい機能の告知は記事ページ側に置き、concept 本文は docs の権威を維持する。
- **個別の人物・組織の専用ページは作らない**。それらは関連する concept / component / channel / provider / article、または該当する `sources/` ページ内で言及する。記事の `author` は記事 frontmatter にプレーン文字列で記録する（人物への `[[wikilink]]` は作らない）。
- 略称（WS, MCP, TUI, CLI, A2UI 等）は専用ページを作らず、対応する正式名称ページ／構成要素ページにリダイレクトする位置づけで `index.md` に併記する。

### 言語ポリシー

- **wiki 内の解説文（sources / concepts / components / channels / providers / articles / questions / overview / index / log）は日本語**で書く。
- 公式ドキュメントは既に日本語（ja-JP）で取り込むため、**原文を一文ずつ訳す翻訳工程は設けない**。`sources/` ページは原典の「翻訳」ではなく「初学者向けの解説・要約」である。
- **例外：非公式の英語記事（`raw/articles/`）だけは翻訳を作る**。英語記事については、(a) 原文を**一文一文を正確に全訳した全文翻訳**（見出し・コードブロック・リンク・箇条書きを保持し、要約・省略・意訳をしない）を `wiki/articles/translations/<slug>.md` に、(b) 既存 wiki と接続した**日本語要約・解説**を `wiki/articles/<slug>.md` に作る。日本語の記事は要約・解説のみ（翻訳不要）。
- 固有名詞・術語は無理に和訳せず、初出時に「Gateway（ゲートウェイ, すべてのメッセージング接続を所有する単一の常駐プロセス）」「WS（WebSocket, 双方向通信トランスポート）」のように原語＋略称＋短い注釈を添える。

### リンク規約

- 内部リンクは Obsidian の `[[wikilink]]` 記法を使う（`[[agent-loop]]` のように slug を直接書く）。
- まだ存在しないページへのリンク（dangling link）も許容する。`lint` 時に未作成ページとして検出する。
- 原典・構成要素・チャネル・プロバイダーへの参照は **ディレクトリ込み**で書く（`[[sources/concepts/architecture]]` `[[components/gateway]]` `[[channels/slack]]` `[[providers/anthropic]]`）。同名スラグの衝突を避けるため。sources はさらにサブフォルダ込みで `[[sources/gateway/protocol]]` のように書く。

---

## 2. Frontmatter 規約

すべての wiki ページに YAML frontmatter を付与する。Obsidian Dataview で集計できるようにする。

**重要（Obsidian 互換）**: frontmatter 内で `[[wikilink]]` を値に使うときは**必ず引用符で囲む**（`"[[...]]"`）。裸の `[[...]]` は YAML では入れ子配列と解釈され、Obsidian で「無効なプロパティ」になる。リンクが複数あるフィールド（`related` / `sources` / `sources_used` / `concepts` / `components`）は **YAML ブロックリスト**（各行 `  - "[[...]]"`）にする。単一リンクは `key: "[[...]]"` と引用符付きの 1 行で書く。本文（frontmatter 外）の `[[wikilink]]` は引用符不要（通常どおり）。

### sources/*.md

```yaml
---
type: source
source_kind: docs                 # docs | article | blog | video
source_url: https://docs.openclaw.ai/ja-JP/concepts/architecture
source_path: raw/docs/concepts/architecture.md
doc_section: concepts             # docs のトップセクション（concepts / gateway / channels / providers / cli ...）
title: "Gateway アーキテクチャ"
ingested: 2026-06-14
tags: [gateway, websocket, architecture, pairing]
related:
  - "[[components/gateway]]"
  - "[[concepts/architecture]]"
---
```

### concepts/*.md

```yaml
---
type: concept
aliases: [Architecture, Gateway Architecture]
tags: [gateway, websocket, runtime]
related:
  - "[[concepts/queue]]"
  - "[[concepts/agent-loop]]"
components:                        # この概念を体現する代表的構成要素（components/）
  - "[[components/gateway]]"
  - "[[components/node]]"
sources:
  - "[[sources/concepts/architecture]]"
updated: 2026-06-14
---
```

### components/*.md

```yaml
---
type: component
aliases: [Gateway, ゲートウェイ]
tags: [daemon, websocket, runtime]
concepts:                         # 属する/関連する上位概念（concepts/）
  - "[[concepts/architecture]]"
  - "[[concepts/pairing]]"
related:
  - "[[components/node]]"
sources:
  - "[[sources/concepts/architecture]]"
  - "[[sources/gateway/protocol]]"
updated: 2026-06-14
---
```

### channels/*.md

```yaml
---
type: channel
aliases: [Slack]
tags: [messaging, channel]
related:
  - "[[components/gateway]]"
  - "[[concepts/channel-routing]]"
sources:
  - "[[sources/channels/slack]]"
updated: 2026-06-14
---
```

### providers/*.md

```yaml
---
type: provider
aliases: [Anthropic, Claude]
tags: [model-provider, llm]
related:
  - "[[concepts/model-providers]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/providers/anthropic]]"
updated: 2026-06-14
---
```

### articles/*.md（非公式記事の日本語要約・解説）

```yaml
---
type: article
source_kind: blog                 # blog | article | video
source_url: https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
source_path: raw/articles/Safer Than YOLO_ Auto Mode for Exec Approvals.md
lang: en                          # en | ja（原文の言語）
author: [Vincent Koc, Jesse Merhi, Josh Avant]   # プレーン文字列。人物への [[wikilink]] は作らない
published: 2026-05-31
summarized: 2026-06-15
title: "YOLO より安全：Exec 承認の auto モード"
tags: [exec, approvals, auto-mode, security]
related:                          # 接続する concept / component / source（引用リンク）
  - "[[concepts/exec]]"
  - "[[concepts/threat-model]]"
translation: "[[articles/translations/safer-than-yolo-auto-mode-for-exec-approvals]]"  # 英語記事のみ
---
```

### articles/translations/*.md（英語記事の全文翻訳）

```yaml
---
type: translation
of: "[[articles/safer-than-yolo-auto-mode-for-exec-approvals]]"
source_url: https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
lang: ja                          # 訳文の言語
translated: 2026-06-15
---
```

### questions/*.md

```yaml
---
type: question
asked: 2026-06-14
question: "OpenClaw でリモートから Gateway に安全に接続するにはどの方法が推奨か？"
sources_used:
  - "[[sources/concepts/architecture]]"
  - "[[sources/gateway/remote]]"
---
```

---

## 3. オペレーション（ingest / query / lint）

主要オペレーションは **skill として切り出してある**。実行時は対応する skill の手順に従うこと（手順の本体は各 SKILL.md にあり、本ファイルには重複させない）。

- **ingest（原典の取り込み）** — `.claude/skills/ingest/SKILL.md`
  Obsidian Web Clipper で `Clippings/` に保存された公式ドキュメントを読み、解説ページ（sources）・関連する概念ページ（concepts）・構成要素ページ（components）・チャネル／プロバイダーページ（channels / providers）を作成／更新し、index/log/overview を更新する。**ingest が完了したら、最後に原典を `raw/docs/<section>/<slug>.md` の適切なフォルダへ移送する**（作業中は clip を `Clippings/` から読む）。**解説・図の具体テンプレと書式もこの skill 内に置いてある**。
  - **非公式記事（ブログ等）の取り込み**は別フロー（同 skill 内「記事の取り込み」節）。入口は `raw/articles/`（Clippings/ を経由しない・移送なし）。英語記事は全文翻訳（`articles/translations/`）＋日本語要約（`articles/`）を作り、二次資料として既存 concept へ引用リンクで接続する。
- **query（質問への回答）** — `.claude/skills/query/SKILL.md`
  index から関連ページを辿り、`[[wikilink]]` 引用付きで回答する。必要なら成果物を `questions/` として保存する。
- **lint（健康診断）** — `.claude/skills/lint/SKILL.md`
  矛盾・古い記述・孤立ページ・dangling link・欠落クロスリファレンス・概念/構成要素/チャネル/プロバイダー間のリンク不備・sources のツリー整合・データギャップを点検し、一覧で提示する。

共通方針：

- ingest は基本 **1 ページずつ**、ユーザーと対話しながら進める（1 件の ingest で 5〜15 ページが触られるのが普通）。
- 下記 §4（「機械的なまとめ」にしないルール）は、**すべてのページ生成および query の回答に常時適用**する。

---

## 4. 「機械的なまとめ」にしない（最重要ルール）

すべての wiki ページ（sources / concepts / components / channels / providers / articles / questions / overview）および query の回答で守ること（**記事の全文翻訳 `articles/translations/` は例外**：要約でなく忠実な全訳なので、略称展開や補足を加えず原文の構造に忠実に訳す。要約ページ `articles/` には本ルールを適用する）：

1. **略称は必ず初出時に展開する**。`WS` → `WS（WebSocket, ブラウザ等と双方向通信するためのトランスポート）` のように、**展開＋短い意味付け**をセットで書く。OpenClaw で頻出する略称の例：WS（WebSocket）, MCP（Model Context Protocol, エージェントに外部ツール・データを接続する規格）, CLI（Command Line Interface）, TUI（Terminal User Interface, 端末上の UI）, A2UI（Agent-to-UI, エージェントが描画する UI ホスト）, LLM（Large Language Model）, TLS, VPN, SSH。
2. **難概念は補足を入れる**。OpenClaw 固有の用語（例: Gateway（すべてのメッセージング接続を所有する単一の常駐プロセス）, Node（`role: node` で WS 接続し camera/screen/location 等のコマンドを公開する端末）, pairing（ペアリング, 新規デバイスの承認とデバイストークン発行）, heartbeat（定期的な生存確認イベント）, presence（在席/接続状態）, agent loop（エージェントの実行サイクル）, queue（コマンドキューと並行制御）, sandboxing（ツール実行の隔離））が出てきたら、その場で 1 文の補足説明を付ける。リンクで概念ページに飛べる場合でも、文脈で必要なら補足を残す。
3. **初学者の読者を想定する**。AI エージェントやメッセージング基盤に詳しくない読者が読んで「何のことを言っているか分からない」段落を作らない。逆に、自明な内容を冗長に説明するのも避ける。
4. **原典の見出しをそのままコピーしない**。解説ページの構造は ingest skill のテンプレートに従い、原典の構造を「再解釈」した形にする。
5. **「なぜ重要か」を自分の言葉で説明する**。ドキュメントの記述をなぞるだけにしない。「この設計／機能が OpenClaw の運用や安全性にとってなぜ効くのか」を 1〜2 文加える。
6. **既存 wiki との接続を明示する**。「WebChat は [[components/gateway]] の WS API を使う静的 UI であり、他クライアントと同じ認証・[[concepts/pairing]] を通る」のように、既存知識（概念・構成要素・チャネル・プロバイダー）と結びつける一文を必ず入れる。

---

## 5. index.md と log.md の運用

### index.md

カテゴリ別の全ページカタログ。1 行 = 1 ページ。フォーマット：

```markdown
- [[<slug>]] — <一行の説明>
```

セクション：
- Overview
- Sources（docs セクション別に分けてよい）
- Concepts
- Components
- Channels
- Providers
- Articles（非公式記事の要約。英語記事は全文翻訳へのリンクも併記）
- Questions

略称リダイレクト（例: `WS → WebSocket`、`MCP → Model Context Protocol`）は対応するセクション内に併記してよい。ingest / query で新規ページを作るたびに必ず更新する。

### log.md

時系列の append-only ログ。

```markdown
## [YYYY-MM-DD] ingest | <タイトル>

- 取り込み: `raw/docs/concepts/architecture.md`（元: https://docs.openclaw.ai/ja-JP/concepts/architecture）
- 作成: [[sources/concepts/architecture]], [[components/gateway]]
- 更新: [[concepts/architecture]], [[overview]], [[index]]
- メモ: ...
```

`grep "^## \[" log.md | tail -10` で直近の動きを追えるよう、必ずこのプレフィックス形式を守る。スキーマ変更は `## [YYYY-MM-DD] schema-update | <要点>` で記録する（§7）。

---

## 6. ツール

- **Obsidian**：wiki の閲覧・グラフビュー確認。ユーザーが裏側で開いている。
- **Obsidian Web Clipper**：公式ドキュメントを markdown 化して `Clippings/` に保存。ingest がそれを `raw/docs/` へ移送する。
- **Marp**：スライド出力が必要な質問への回答に使う（任意）。
- **Dataview**：frontmatter ベースの動的集計（任意）。
- **Mermaid**：Obsidian はコードフェンス ```mermaid を図としてレンダリングする。公式ドキュメントの構成図・シーケンス図は Mermaid で描き直すとよい（§ingest skill）。

検索は規模が小さいうちは index.md ベースで十分。ページ数が増えてきたら検索系プラグインの導入を検討する。

---

## 7. このスキーマ自体について

このスキーマはユーザーと共進化する。運用していく中で「このカテゴリが必要」「このルールは緩めたい」となったら、ユーザーに提案してこの CLAUDE.md（およびオペレーション手順を持つ `.claude/skills/` 配下の SKILL.md）を更新する。スキーマ変更は log.md にも `## [YYYY-MM-DD] schema-update | <要点>` として記録する。
