---
name: ingest
description: OpenClaw LLM Wiki への原典取り込み。ユーザーが Obsidian Web Clipper で公式ドキュメントを Clippings/ に保存して「ingest して」「取り込んで」と指示したときに使う。Clippings/ の clip を読んで解説ページ（sources）・概念ページ（concepts）・構成要素ページ（components）・チャネル／プロバイダーページ（channels / providers）を作成／更新し、index.md・log.md・overview.md を更新したうえで、最後に原典を raw/ の適切なフォルダ（docs/<section>/）へ移送する。解説・図の具体テンプレと書式もこの skill 内にある。
tools: Read, Write, Edit, Bash, WebFetch, WebSearch
---

# ingest — 原典の取り込み

OpenClaw wiki に新しい原典（主に公式ドキュメント `https://docs.openclaw.ai/ja-JP/...`）を取り込む。スキーマ全体は `CLAUDE.md` を、frontmatter 規約は CLAUDE.md §2 を、横断的な品質ルール（略称展開・難概念補足・既存 wiki との接続など）は **CLAUDE.md §4「機械的なまとめにしない」を必ず参照**すること。本 skill は手順と、解説・図の具体テンプレを持つ。

公式ドキュメントは既に日本語（ja-JP）なので、**原文を一文ずつ訳す翻訳工程は無い**。`sources/` ページは原典の「翻訳」ではなく「初学者向けの解説・要約」である。

## 標準フロー

ユーザーが `Clippings/` に新しいドキュメントを保存し「これを ingest して」と指示したときの標準フロー：

1. **取り込み対象の特定**：`Clippings/` にある対象 clip を読み、frontmatter の `source:` URL（例 `https://docs.openclaw.ai/ja-JP/concepts/architecture`）から、`ja-JP/` を除いた **`<section>/<slug>`**（例 `concepts/architecture`）を導く。スラグは URL 末尾セグメント。これが移送先 **`raw/docs/<section>/<slug>.md`**（後述ステップ 10 でアーカイブする最終置き場）になる。**この時点ではまだ移動しない**。ingest の作業中は clip を `Clippings/` から読み、frontmatter の `source_path` や「原典」行には最終置き場 `raw/docs/<section>/<slug>.md` を書く。
2. **読解**：clip を読み、主要な主張・仕組み・設定・前提・注意点を把握する。図（多くは Mermaid のインライン SVG）や本文中の関連リンクも確認する（後述「図・画像の扱い」を参照）。
3. **対話**：ユーザーに重要な発見・疑問点・既存 wiki との関係（既知概念との接続、矛盾、新概念の発生）を簡潔に伝え、強調すべき観点をすり合わせる。
4. **解説ページ（sources）の作成**：`wiki/sources/<section>/<slug>.md` を作成する（`mkdir -p wiki/sources/<section>` を先に）。詳細は下記「解説ページの仕様」を参照。**機械的な要約にしない**。略称の定義、難概念の補足、初学者でも読める平易な説明を心がける（CLAUDE.md §4 参照）。
5. **概念 / 構成要素 / チャネル / プロバイダーページの追加・更新**：原典で重要な役割を果たす項目について、対応するディレクトリにページを作るか既存ページを更新する（CLAUDE.md §1 の命名規約）。
   - **概念（concepts/）**：横断的な概念・仕組み（例: `architecture`, `agent-loop`, `memory`, `queue`, `pairing`, `channel-routing`, `model-providers`）。スラグは英語名。
   - **構成要素（components/）**：landmark な具体的サブシステムは専用ページを作ってよい（例: `gateway`, `node`, `webchat`, `cli`, `clawhub`）。**component ページは必ず関連する上位概念ページへリンクする**（frontmatter の `concepts` と本文の双方）。逆に概念ページからは代表 component として `[[components/...]]` へリンクする。内部構造・設定・限界は component ページ側に、俯瞰と位置づけは概念ページ側に書く。専用ページにするほど確立していない要素は概念ページ内の記述に留めてよい。
   - **チャネル（channels/）・プロバイダー（providers/）**：1 統合 = 1 ページ。**関連する構成要素（多くは `[[components/gateway]]`）と概念（channels なら `[[concepts/channel-routing]]`、providers なら `[[concepts/model-providers]]`）へリンクする**（frontmatter の `related` と本文の双方）。
   - 人物・組織の専用ページは作らない（関連 concept / component / channel / provider / source 内で言及）。
   - **どのページも初学者向けの丁寧な解説を心がける**（CLAUDE.md §4）。
6. **クロスリファレンスの更新**：関連する既存ページの「関連ページ」セクション、frontmatter の `sources` / `related` / `concepts` / `components` を更新する。
7. **overview.md の更新**：必要に応じて OpenClaw 全体の総括に変化を反映する（新カテゴリ・大きな構成要素の追加等）。
8. **index.md の更新**：新規作成したすべてのページをカテゴリ別に追記する（CLAUDE.md §5）。Sources は docs セクション別に並べてよい。
9. **log.md への追記**：`## [YYYY-MM-DD] ingest | <タイトル>` のフォーマットで、何を取り込み・どのページを作成/更新したかを箇条書きで記録する（CLAUDE.md §5）。取り込み行には元の URL も併記する。
10. **原典のアーカイブ（移送）**：wiki ページと index/log/overview の更新がすべて完了したら、`mkdir -p raw/docs/<section>` のうえで `Clippings/<title>.md` を `raw/docs/<section>/<slug>.md` へ **移動・改名**する。これで docs のセクション階層が raw 側に再現され、`Clippings/` は次の clip 用に空に戻る。公式ドキュメント以外の記事・ブログを取り込んだ場合は、適切な置き場（`raw/articles/`）へ移送する。移送後、対象ファイルが `Clippings/` に残っていないことを確認する。

1 件の ingest で 5〜15 ページが触られるのが普通。基本は **1 ページずつ**取り込み、ユーザーと対話しながら進める。原典の移送（ステップ 10）は **ingest の最後**に行う（作業中は clip を `Clippings/` から読む）。

---

## 記事の取り込み（`raw/articles/`）

公式ドキュメントではない **非公式のブログ記事等**を取り込むときの専用フロー。docs フローとの主な違いは「入口が `raw/articles/`（Clippings/ を経由しない）」「移送ステップが無い（原典は既に最終置き場にある・immutable）」「英語記事は全文翻訳を作る」「二次資料として扱う（公式 docs を上書きしない）」。

1. **対象の特定**：`raw/articles/` に置かれた記事を読む。frontmatter の `source:` URL（例 `https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals`）末尾から **slug** を導く。原典は `raw/articles/<元のファイル名>.md` のまま（リネーム不要・触らない）。
2. **言語判定**：原文が**英語か日本語か**を判定する。英語 → 全文翻訳＋要約。日本語 → 要約のみ（翻訳不要）。
3. **全文翻訳の作成（英語記事のみ）**：`wiki/articles/translations/<slug>.md` を作る（`mkdir -p wiki/articles/translations` を先に）。frontmatter は CLAUDE.md §2 の `articles/translations/*.md`。本文は **原文を一文一文を正確に全訳**する：見出し・コードブロック（中身は訳さずそのまま）・リンク（リンク先 URL は保持。本文中の docs.openclaw.ai リンクは対応する wiki ページの併記を任意で添えてよい）・箇条書き・表の構造を**保持**し、**要約・省略・意訳をしない**（§4 の「機械的なまとめにしない」ルールはここには適用しない＝忠実訳が目的）。冒頭に `> 原典: raw/articles/<file> ・ <source_url>` と `> 要約は [[articles/<slug>]]` を置く。
4. **要約・解説の作成**：`wiki/articles/<slug>.md` を作る。frontmatter は CLAUDE.md §2 の `articles/*.md`（`author` はプレーン文字列・人物 wikilink を作らない）。本文は「解説ページの仕様」テンプレに準じた**日本語の要約＋位置づけ＋既存 wiki との接続**（§4 を適用）。英語記事なら冒頭で全文翻訳 `[[articles/translations/<slug>]]` へリンク。記事が告知する**公式 docs より新しい機能**は、ここ（記事ページ）に書く。
5. **二次資料としての接続（引用リンクのみ）**：関連する concept / component / source へ記事側からリンクし、逆に**それらの「関連」セクションに「ブログで〜が告知 → [[articles/<slug>]]」の引用リンクを 1 行だけ追加**する。**concept / component の本文は書き換えない**（公式 docs の権威を維持）。矛盾がある場合は概念側を直さず、記事側に「ブログ時点の情報」と注記する。
6. **index / log の更新**：`wiki/index.md` の `## Articles` に追記。`wiki/log.md` に `## [YYYY-MM-DD] ingest | articles | <タイトル>` を追記（source URL・作成ページ・接続先 concept を箇条書き）。overview は原則変更しない（記事は補助資料）。
7. **移送はしない**：原典 `raw/articles/<file>.md` はそのまま。`raw/` には書き込まない。

検証の要点：要約→翻訳→concept の導線が貼れていること、翻訳が原文の全見出し・全コードブロックを含むこと、concept 本文が書き換わっていないこと、`author` の人物 wikilink が無いこと。

---

## 解説ページ（wiki/sources）の仕様

解説ページは、原典を「読まなくても本質が掴め、かつ深く知りたくなったら原典に戻れる」状態を目指す。以下のセクションを基本構成とする（必要に応じて増減）。**CLAUDE.md §4 の品質ルールを必ず守る**：

```markdown
---
<frontmatter — CLAUDE.md §2 の sources/*.md を参照>
---

# <タイトル>

> 原典: `raw/docs/<section>/<slug>.md` ・ <source_url>

## 一言まとめ

（1〜2 文で「このページが何を説明しているか」）

## 位置づけ

（OpenClaw のどの構成要素・概念の話か。全体像の中でどこに当たるか。**既存 wiki の concept / component / channel / provider と結びつける**）

## 仕組み・ふるまい

（どう動くのか。データフロー・ライフサイクル・主要イベントなど。図があると望ましい＝下記「図・画像の扱い」）

## 設定・使い方の要点

（設定キー・コマンド・前提条件など、実際に使う/運用するうえでの要点）

## 注意点・落とし穴

（セキュリティ上の注意、不変条件、よくある誤解、未対応事項など）

## 用語と略称

（本ページに出てきた略称をすべて展開・定義する。例: WS = WebSocket、MCP = Model Context Protocol、TUI = Terminal User Interface）

## 関連ページ

- [[concepts/...]]
- [[components/...]]
- [[channels/...]] / [[providers/...]]
- [[sources/...]]
```

---

## 図・画像の扱い

### 公式ドキュメントの図（Mermaid インライン SVG）

公式ドキュメントのシーケンス図・構成図は、Web Clipper で取り込むと **巨大なインライン SVG の塊**として markdown 内に混入する（実例: `raw/docs/concepts/architecture.md` の接続ライフサイクル図）。これは可読性が無く、そのまま wiki に貼らない。次の方針で扱う：

1. **SVG はそのまま保存・引用しない**。読解時は SVG 内の `<text>` ラベル（`req:connect`, `res (ok)`, `event:presence` 等）やメッセージの向きを読み取って、図の意味を把握する。
2. **解説ページ（sources）では Mermaid コードフェンスで描き直す**。Obsidian は ```mermaid フェンスを図としてレンダリングする。原図の構造（参加者・矢印・順序）を再構成して記述する：

   ````markdown
   ```mermaid
   sequenceDiagram
     participant Client
     participant Gateway
     Client->>Gateway: req:connect
     Gateway-->>Client: res (ok) / hello-ok
     Gateway-->>Client: event:presence
     Client->>Gateway: req:agent
     Gateway-->>Client: event:agent (streaming)
   ```
   ````

3. **再構成が難しい / 図が本質的でない場合**は、Mermaid を諦めて要点を箇条書き・文章で説明してよい。**原図と食い違う図を創作しない**（読み取れた範囲に忠実に）。

### スクリーンショット・ラスター画像

ドキュメントに UI のスクリーンショットや図版（PNG/JPEG）が含まれ、本文理解に必要な場合：

- 画像を `raw/assets/<source-slug>/` にローカル保存する（`curl -A "Mozilla/5.0" <URL>` 等。CDN の変換 URL は素のオリジナルに直す）。`<source-slug>` は `<section>-<slug>`（例 `concepts-architecture`）。
- 保存後は解説ページから **ローカルパス**（`../../raw/assets/<source-slug>/<name>`）を参照し、元 URL は残さない。`file` コマンドで妥当な画像データか確認する。
- サイト UI（ロゴ・ナビ・装飾カバー等）や、画像でない埋め込み（動画リンク等）は保存しない。動画等は解説ページに参照リンクとして記載する。
- 取得できなかった画像は log.md のメモに「取得失敗: <URL>」として残す。

### キャプション書式（Obsidian 対応）

Obsidian は Markdown 標準の alt テキストをキャプション表示しない。Reading view にキャプションを出すため、**HTML の `<figure>` + `<figcaption>` で Markdown 画像記法を包む**：

```markdown
<figure>

![](../../raw/assets/<source-slug>/figN.png)

<figcaption>図N: ここにキャプション本文。</figcaption>
</figure>
```

ルール：
- `<figure>` の**直後と `</figure>` の直前に空行を入れる**。`<figcaption>` の**前にも空行を入れる**。これがないと Obsidian が中の Markdown 画像を解釈しない。
- 画像パスは Markdown 記法 `![](path)` を維持する（`<img src="">` は使わない）。
- `<figcaption>` 内では `$...$` などのインライン LaTeX を使わない（Unicode 添字や簡易表記で代替）。
- 表（Markdown テーブル）には `<figure>` を使わず、テーブル直上に `**表N**: ...` の太字キャプションを置く。
