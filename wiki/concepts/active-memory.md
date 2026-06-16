---
type: concept
aliases: [Active Memory, アクティブメモリ]
tags: [active-memory, memory, recall, subagent, plugin]
related:
  - "[[concepts/memory]]"
  - "[[concepts/memory-search]]"
  - "[[concepts/system-prompt]]"
  - "[[concepts/session]]"
sources:
  - "[[sources/concepts/active-memory]]"
updated: 2026-06-14
---

# Active Memory

**Active Memory** は、対象セッションで**メイン応答の前に走る Plugin 所有のブロッキングなメモリサブエージェント**。普通のメモリは「エージェントが検索を決める／ユーザーが頼む」リアクティブなものだが、Active Memory はメイン応答が出る前に**先回りで関連メモリを表面化する**1 機会を与える。

## なぜ重要か

「メモリが応答を自然に感じさせるはずだった瞬間」は、リアクティブ検索ではすでに過ぎている――Active Memory はそのギャップを埋める。[[concepts/memory-search]] を応答前のプロアクティブ想起として使い、結果を隠れたプロンプト接頭辞（[[concepts/system-prompt]] の Active Memory セクション）として注入する。**返信経路で同期実行されるためレイテンシが直結する**のが設計上の最重要トレードオフで、`queryMode`（message/recent/full）・専用の高速モデル・タイムアウトでバランスを取る。

## 押さえる点

- 実行ゲート：プラグイン有効 ＋ エージェント対象 ＋ 許可チャットタイプ ＋ 対話的な永続チャット。ヘッドレス/Heartbeat/サブエージェントでは走らない。
- 注入は「信頼できないコンテキスト（指示として扱うな）」扱い。`/verbose on`・`/trace on` で可視化。
- 想起ツールは設定されたもののみ（既定 `memory_search`/`memory_get`、LanceDB は `memory_recall`）。弱ければ `NONE`。
- コールドスタートは `setupGraceTimeoutMs` 要注意（再起動直後の最初の想起がタイムアウトしやすい）。

設定・クエリモード・プロンプトスタイル・フロー図は [[sources/concepts/active-memory]]。

## 関連

- [[concepts/memory]] / [[concepts/memory-search]] / [[concepts/system-prompt]]
- [[concepts/context-engine]] / [[concepts/session]]
- 📝 ブログ（二次資料・外部）：応答前に関連メモリを system prompt へ注入する設計と「メモリは助言的・指示として扱うな」の優先順位ルール → [[articles/context-engineering-personalization]]
