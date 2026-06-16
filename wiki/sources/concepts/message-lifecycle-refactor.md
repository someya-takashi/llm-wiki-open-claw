---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/message-lifecycle-refactor
source_path: raw/docs/concepts/message-lifecycle-refactor.md
doc_section: concepts
title: "メッセージライフサイクルのリファクタリング"
ingested: 2026-06-14
tags: [refactor, message-lifecycle, durability, sdk, design, internal]
related:
  - "[[concepts/messages]]"
  - "[[concepts/streaming]]"
  - "[[concepts/retry]]"
---

# メッセージライフサイクルのリファクタリング（解説）

> 原典: `raw/docs/concepts/message-lifecycle-refactor.md` ・ https://docs.openclaw.ai/ja-JP/concepts/message-lifecycle-refactor
>
> ℹ️ これは**実装済みの仕様ではなく、目標とする設計（リファクタリング RFC）**。散在するチャネルターン/返信ディスパッチ/プレビューストリーミング/送信ヘルパーを、1 つの耐久的なメッセージライフサイクルへ置き換えるためのコントリビューター向け設計ドキュメント。

## 一言まとめ

「reply（返信）」中心だった現在のチャネルスタックを、**receive（受信）と send（送信）** を核プリミティブとする 1 つの耐久的ライフサイクルに作り変える目標設計。返信は送信メッセージ上の「関係」に過ぎず、送信・受信は明示的なトランザクション境界（begin → commit → fail）を持つ。

## 位置づけ

[[concepts/messages]]・[[concepts/streaming]]・[[concepts/retry]] の現在の挙動が「なぜそう散らばっているか」と「どこへ向かうか」を説明する設計の地図。利用者向け機能ではなく、**信頼性（最終返信を失わない）と Plugin SDK の単純化**を狙う内部リファクタリング。

## 仕組み・ふるまい（目標設計）

### 動機となった信頼性バグ

`Telegram の更新を ack → アシスタント最終テキストは存在 → sendMessage 成功前にプロセス再起動 → 最終返信が消失`。目標の不変条件：コアが「可視の送信メッセージが存在すべき」と判断したら、**プラットフォーム送信を試みる前にその意図を耐久化し、成功後に受領をコミット**する。これで少なくとも 1 回のリカバリーを得る。

### コアモデル（4 つの名前空間）

`core.messages.receive`（インバウンド）/`send`（アウトバウンド）/`live`（プレビュー・編集・進捗・ストリーム状態）/`state`（耐久化された意図ストレージ・受領・冪等性・リカバリー・ロック・重複排除）。`receipt`（受領）を第一級にし、再起動後の編集/削除/重複抑制/リカバリーを駆動する。

### 危険な境界とリカバリー

最も危ういのは**プラットフォーム成功後・receipt コミット前**にプロセスが落ちる地点。アダプターがネイティブ冪等性や調整（`reconcileUnknownSend`）を提供しない限り、その試行は盲目的に再実行せず `unknown_after_send` で再開する。耐久性ポリシーは明示的：`required`（書けなければ fail closed）/`best_effort`/`disabled`（移行中の既定）。

## 設定・使い方の要点（移行方針）

- **互換性ガードレール**：移行中、汎用の耐久配信はオプトイン。`channel.turn.*` やレガシー互換ヘルパーは既定で非耐久（direct-send）のまま。`durable: undefined` は「耐久でない」を意味する。
- 公開 SDK は `openclaw/plugin-sdk/channel-message` 1 サブパスへ収れんし、`reply-*` 系ヘルパーは互換ラッパーとして deprecate。
- 8 フェーズの移行計画：内部ドメイン型 → 耐久送信コア → channel-turn ブリッジ → prepared dispatcher ブリッジ → 統一ライブライフサイクル → 公開 SDK → 全送信元（cron/hook/承認/サブエージェント等も `messages.send` へ）→ `channel.turn` の非推奨化。

## 注意点・落とし穴

- **現在の挙動ではなく最終状態の記述**。移行中は非耐久パスにフォールスルーしてよい。
- チャネル固有の送信後副作用（self-echo 抑制キャッシュ・参加スレッドマーカー・モデル署名・ネイティブ編集アンカー）を汎用配信が迂回すると壊れる――先にアダプター/フックへ移すこと（iMessage の echo-cache、Tlon の署名等が具体例）。
- **OpenClaw 起源のメタデータ**（`MessageOrigin`）：Gateway 障害出力は人間には見せつつ、`allowBots` 有効な共有ルームでは**タグベースで**bot エコーを破棄する（可視テキストの接頭辞フィルターで実装しない）。

## 用語と略称

- **RFC** = Request for Comments（設計提案ドキュメント）
- **receive / send / live / state** = 受信 / 送信 / ライブ表示 / 耐久状態の 4 コア
- **receipt（受領）** = 送信結果（プラットフォーム ID 群）を表す第一級オブジェクト
- **idempotency key（冪等性キー）** = 重複送信を防ぐ識別子
- **fail closed** = 条件を満たせないとき安全側（実行しない）に倒す
- **`unknown_after_send`** = 送信後に結果不明となった状態（盲目再実行しない）

## 関連ページ

- [[concepts/messages]] / [[concepts/streaming]] / [[concepts/retry]] / [[concepts/progress-drafts]]
- [[concepts/queue]]
