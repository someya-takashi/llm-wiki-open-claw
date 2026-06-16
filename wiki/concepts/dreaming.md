---
type: concept
aliases: [Dreaming, ドリーミング]
tags: [dreaming, memory, consolidation, promotion]
related:
  - "[[concepts/memory]]"
  - "[[concepts/memory-search]]"
  - "[[concepts/commitments]]"
  - "[[concepts/heartbeat]]"
sources:
  - "[[sources/concepts/dreaming]]"
updated: 2026-06-14
---

# Dreaming

**Dreaming** は `memory-core` の**バックグラウンド記憶統合システム**――強い短期シグナルを耐久的な長期メモリ（`MEMORY.md`）へ、説明可能・レビュー可能なやり方で昇格する。睡眠になぞらえた 3 フェーズ（Light＝整理／Deep＝昇格／REM＝内省）で動き、**オプトイン（既定無効）**。

## なぜ重要か

[[concepts/memory]] の「日次ノート → 長期メモリ」の手作業をなくし、**長期メモリのシグナル品質を高く保つ**ための仕組み。Deep フェーズだけが重み付きスコア（頻度・関連性・クエリ多様性・新しさ・統合・概念的豊かさ）としきい値ゲートを通った候補を `MEMORY.md` に書き、`DREAMS.md` の Dream Diary は**レビュー用であって昇格ソースではない**。これにより「何を永続化したか」を人間が後追いできる。

## 押さえる点

- 有効化：`plugins.entries.memory-core.config.dreaming.enabled: true`。スイープは Cron（既定 `0 3 * * *`）で自動管理され、配信は Heartbeat に依存（`status: blocked` は Heartbeat 未発火のサイン）。
- CLI：`openclaw memory promote [--apply]`・`promote-explain`・`rem-harness`（プレビュー）。
- 根拠付きバックフィル：履歴ノートを再生して短期ストアへステージング（`MEMORY.md` はディープ昇格でのみ書かれる）。

フェーズ詳細・ランキング重み・Dreams UI は [[sources/concepts/dreaming]]。

## 関連

- [[concepts/memory]] / [[concepts/memory-search]] / [[concepts/commitments]]
- 📝 ブログ（二次資料・外部）：セッションメモリを重複排除・競合解決・忘却しつつ長期へ昇格する「統合（consolidation）」パターン（Dreaming とほぼ同型）→ [[articles/context-engineering-personalization]]
