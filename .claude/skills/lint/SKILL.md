---
name: lint
description: OpenClaw LLM Wiki の健康診断。ユーザーが「lint して」「wiki を点検して」と指示したときに使う。wiki 全体を走査し、矛盾・古い記述・孤立ページ・未作成ページ（dangling link）・欠落クロスリファレンス・概念/構成要素/チャネル/プロバイダー間のリンク不備・sources のツリー整合・データギャップを検出して一覧で提示する。
tools: Read, Glob, Grep, Bash, WebFetch, WebSearch
---

# lint — wiki の健康診断

ユーザーが「lint して」と指示したら、wiki 全体を点検する。スキーマ規約は `CLAUDE.md` を参照。

## 点検観点

- **矛盾の検出**：同じ概念について異なる主張をしているページの組。
- **古い記述**：新しいドキュメントで更新されたはずなのに古い値や主張が残っているページ。
- **孤立ページ**：他のどのページからもリンクされていないページ。
- **未作成ページ**：本文中で言及されている（`[[...]]` で張られている）が対応ページが存在しない項目（dangling link）。`grep` で `[[...]]` を集計し、実ファイルと突き合わせる。
- **欠落クロスリファレンス**：明らかに相互参照すべきページ同士でリンクが張られていない箇所。
- **概念/構成要素/チャネル/プロバイダー間のリンク不備**（CLAUDE.md §1 の命名規約）：
  - 構成要素ページ（components/）が関連する上位概念ページ（concepts/）へリンクしているか（frontmatter `concepts` と本文の双方）。逆に概念ページから代表 component `[[components/...]]` への言及があるか。
  - チャネル（channels/）・プロバイダー（providers/）ページが関連する構成要素（多くは `[[components/gateway]]`）と概念（`[[concepts/channel-routing]]` / `[[concepts/model-providers]]` 等）へリンクしているか（frontmatter `related`）。
  - 抽象概念ページに統合すべき細かい話題が乱立していないか／逆に確立した landmark 構成要素が概念ページ内に埋もれたまま独立ページ化されていないか（粒度のバランス）。
  - 人物・組織の専用ページが誤って作られていないか。
  - 人物・組織の専用ページが誤って作られていないか（記事の `author` が人物 `[[wikilink]]` になっていないかを含む）。
- **sources のツリー整合**：`wiki/sources/<section>/<slug>.md` のサブフォルダ構造が `raw/docs/<section>/<slug>.md`（= docs の URL ツリー）と対応しているか。命名ずれ・取り違え・raw に対応原典が無い sources を検出する。
- **articles のミラー整合**（非公式記事）：`wiki/articles/<slug>.md` が `raw/articles/<file>.md` と 1:1 対応しているか（原典に対応する要約が無い／要約に対応する原典が無いを検出）。英語記事には `wiki/articles/translations/<slug>.md` があるか。**`comm` 等のミラー検査では `wiki/sources ↔ raw/docs` と `wiki/articles ↔ raw/articles` を別系統として扱う**（articles を docs ミラーに混ぜない）。
- **記事＝二次資料の整合**：記事ページが既存 concept / component を引用リンクで接続しているか。記事の告知内容が公式 docs ベースの concept 本文を**上書きしていないか**（concept 本文は docs 権威・記事は引用のみ、という分離が保たれているか）。記事と docs が矛盾する場合に concept 側が記事に引きずられていないか。
- **データギャップ**：公式ドキュメントや Web 検索で埋められそうな情報の欠落。

## 進め方

1. `wiki/` 配下（sources / concepts / components / channels / providers / questions / overview / index）と `raw/docs/` を Glob / Grep で走査し、frontmatter・`[[wikilink]]`・見出し・サブフォルダ構造を収集する。
2. 上記観点ごとに検出結果を一覧化してユーザーに提示し、優先度を相談する。
3. 修正そのものは行わず、別途 ingest / query フロー（対応する skill）で行う。lint は検出と提示に専念する。
