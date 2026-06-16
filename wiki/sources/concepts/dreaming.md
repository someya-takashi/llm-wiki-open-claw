---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/dreaming
source_path: raw/docs/concepts/dreaming.md
doc_section: concepts
title: "Dreaming"
ingested: 2026-06-14
tags: [dreaming, memory, consolidation, promotion, cron, heartbeat]
related:
  - "[[concepts/dreaming]]"
  - "[[concepts/memory]]"
  - "[[concepts/memory-search]]"
---

# Dreaming（解説）

> 原典: `raw/docs/concepts/dreaming.md` ・ https://docs.openclaw.ai/ja-JP/concepts/dreaming

## 一言まとめ

Dreaming は `memory-core` の**バックグラウンド記憶統合システム**――強い短期シグナルを耐久的な長期メモリ（`MEMORY.md`）へ、説明可能かつレビュー可能なやり方で移す。睡眠になぞらえた 3 フェーズ（Light/Deep/REM）で動き、**オプトイン（既定無効）**。

## 位置づけ

[[concepts/memory]] の「日次ノート → 長期メモリ」の昇格を自動化する仕組み。人間が `MEMORY.md` を手編集しなくても、システムが何を永続化すべきかを判断・提示する。昇格判断は [[concepts/memory-search]] の想起品質シグナルにも依存し、配信は Heartbeat（gateway/heartbeat）に乗る。

## 仕組み・ふるまい

### 3 フェーズモデル（内部実装、ユーザー設定の「モード」ではない）

| フェーズ | 目的 | 耐久書き込み |
| --- | --- | --- |
| **Light** | 最近の短期素材を整理・重複除去・ステージング | いいえ |
| **Deep** | 耐久候補をスコアリングして昇格 | はい（`MEMORY.md`） |
| **REM** | テーマや繰り返しを内省 | いいえ |

各スイープは Light → REM → Deep の順に実行。Deep だけが `MEMORY.md` に追記し、`DREAMS.md` に `## Deep Sleep` 要約を書く。

### Deep ランキングのシグナル（重み付き）

頻度 0.24／関連性 0.30／クエリ多様性 0.15／新しさ 0.15／統合 0.10／概念的豊かさ 0.06。昇格には `minScore`・`minRecallCount`・`minUniqueQueries` のゲートを通過する必要があり、書き込み前にライブ日次ファイルからスニペットを再取得（古い/削除済みはスキップ）。

### 出力先と Dream Diary

- **マシン状態**：`memory/.dreams/`（リコールストア、フェーズシグナル、ロック）。
- **人間可読**：`DREAMS.md`（物語形式の Dream Diary）と任意の `memory/dreaming/<phase>/YYYY-MM-DD.md`。Dream Diary は**レビュー用であって昇格ソースではない**（昇格できるのは根拠あるメモリスニペットのみ）。

## 設定・使い方の要点

- 有効化：`plugins.entries.memory-core.config.dreaming.enabled: true`。スイープ周期は `dreaming.frequency`（既定 `0 3 * * *`）、`timezone` も設定可。`memory-core` が完全スイープ用の Cron を 1 つ自動管理する。
- スラッシュ：`/dreaming status|on|off`。CLI：`openclaw memory promote [--apply]`、`memory promote-explain "query"`（昇格理由を説明）、`memory rem-harness`（書き込まずプレビュー）。
- `dreaming.model`（Dream Diary サブエージェントの上書き）には `memory-core.subagent.allowModelOverride: true` が必要。
- **根拠付きバックフィル**：`memory rem-backfill` で履歴ノートを再生し短期ストアへステージング（`MEMORY.md` は通常のディープ昇格でのみ書かれる）。Control UI の Dreams タブで確認・消去できる。

## 注意点・落とし穴

- **`Dreaming status: blocked`**：管理 Cron はあるが既定エージェントの Heartbeat が発火していない。Heartbeat が有効で `target` が `none` でないことを確認し、次の間隔後に `openclaw memory status --deep` を再実行。
- フェーズポリシー・しきい値・ストレージ動作は内部実装の詳細（ユーザー向け設定ではない。完全な設定キーは memory-config リファレンス）。

## 用語と略称

- **Dreaming** = 短期シグナルを長期メモリへ昇格するバックグラウンド統合
- **昇格（promotion）** = 候補を `MEMORY.md` に書き込むこと
- **Light / Deep / REM** = 整理 / 昇格 / 内省の 3 フェーズ
- **根拠付きバックフィル** = 履歴ノートを再生して昇格候補をステージングする可逆レーン

## 関連ページ

- [[concepts/dreaming]] — 対応する概念ページ
- [[concepts/memory]] / [[concepts/memory-search]]
