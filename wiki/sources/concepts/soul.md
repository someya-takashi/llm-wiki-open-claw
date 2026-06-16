---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/soul
source_path: raw/docs/concepts/soul.md
doc_section: concepts
title: "SOUL.md パーソナリティガイド"
ingested: 2026-06-14
tags: [soul, persona, prompt, workspace]
related:
  - "[[concepts/soul]]"
  - "[[concepts/agent-workspace]]"
  - "[[concepts/system-prompt]]"
---

# SOUL.md パーソナリティガイド（解説）

> 原典: `raw/docs/concepts/soul.md` ・ https://docs.openclaw.ai/ja-JP/concepts/soul

## 一言まとめ

`SOUL.md` は「エージェントの声が宿る場所」――トーン・意見・簡潔さ・ユーモア・境界線を書くワークスペースファイル。通常セッションで毎ターン注入され効きが大きいので、エージェントの話し方が平板・曖昧・企業的なら真っ先に直すべきファイル、という実践ガイド。

## 位置づけ

[[concepts/agent-workspace]] の標準ファイルの 1 つ `SOUL.md` を深掘りした「書き方」ページ。運用ルール（手順・規約）は `AGENTS.md`、**声・姿勢・スタイルは `SOUL.md`** と役割を分ける。注入の仕組みは [[concepts/system-prompt]]、テンプレは reference/templates/SOUL。

## 仕組み・ふるまい（何を書くか）

- **入れるもの**：トーン、意見、簡潔さ、ユーモア、境界線、デフォルトの率直さ。
- **入れないもの**：人生の物語、変更履歴、セキュリティポリシーの羅列、行動に影響しない「雰囲気だけの壁」。
- 原則：**短い＞長い、鋭い＞曖昧**。
- 良いルールの例：「見解を持つ」「埋め草を省く」「合うときは面白く」「悪いアイデアは早めに指摘」「本当に有用なとき以外は簡潔に」。悪い例：「常にプロフェッショナリズムを維持」「包括的で思慮深い支援を提供」――こうした漠然ルールがぼんやりした人格を生む。

## 設定・使い方の要点

- 原典は「Molty プロンプト」――`SOUL.md` を読ませて意見・簡潔さ・ユーモアを足す方向に書き換えさせるプロンプト例を提供している。ワークスペースではパスは固定で、`SOUL.md` をそのまま参照する。
- 設計上の裏づけ：高レベルの振る舞い・トーン・目標は、ユーザーターンに埋めず**高優先度の指示レイヤー**に置くべき、というプロンプト指針に沿う。OpenClaw ではその層が `SOUL.md` に当たる。安定した人格には、簡潔でバージョン管理された状態を保つ。

## 注意点・落とし穴

- **人格は雑にやってよい許可ではない**。共有チャンネル・公開返信・顧客向けの場では、そのトーンが場に合っているか確認する（「鋭い」は良いが「うっとうしい」は別）。

## 用語と略称

- **ペルソナ** = エージェントの話し方・態度の人格設定
- **ブートストラップ注入** = ワークスペースファイルを毎ターンのシステムプロンプトへ差し込むこと

## 関連ページ

- [[concepts/soul]] — 対応する概念ページ
- [[concepts/agent-workspace]] — `SOUL.md` を含むワークスペース全体
- [[concepts/system-prompt]] — `SOUL.md` がプロンプトに合成される仕組み
