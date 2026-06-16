---
type: concept
aliases: [SOUL.md, persona, パーソナリティ]
tags: [soul, persona, prompt, workspace]
related:
  - "[[concepts/agent-workspace]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/agent]]"
sources:
  - "[[sources/concepts/soul]]"
updated: 2026-06-14
---

# SOUL.md（人格）

`SOUL.md` は **エージェントの声が宿るワークスペースファイル**――トーン・意見・簡潔さ・ユーモア・境界線を書く場所。通常セッションで毎ターン注入されるため効きが大きく、話し方が平板・曖昧・企業的なら最初に直すべきファイル。

## なぜ重要か

OpenClaw は「振る舞い・トーン・目標は高優先度の指示レイヤーに置く」というプロンプト設計に沿い、その層を `SOUL.md` に割り当てている。**運用ルールは `AGENTS.md`、声・姿勢・スタイルは `SOUL.md`** と役割分担することで、人格を安定して反復・バージョン管理できる。原則は「短い＞長い、鋭い＞曖昧」。漠然とした「常にプロフェッショナルに」式のルールは、ぼんやりした人格しか生まない。

詳細（入れる/入れないもの、Molty プロンプト、注意点）は [[sources/concepts/soul]]。

## 位置づけ

- [[concepts/agent-workspace]] の標準ファイルの 1 つ。
- 注入の仕組みは [[concepts/system-prompt]]（内部フック `agent:bootstrap` で差し替えも可能）。
- 共有/公開の場ではトーンが場に合うか確認する（「鋭い」と「うっとうしい」は別）。

## 関連

- [[concepts/agent-workspace]] / [[concepts/system-prompt]] / [[concepts/agent]]
