---
type: concept
aliases: [Messages, メッセージ, message pipeline]
tags: [messages, pipeline, dedup, debounce, silent-reply]
related:
  - "[[concepts/queue]]"
  - "[[concepts/streaming]]"
  - "[[concepts/retry]]"
  - "[[concepts/session]]"
  - "[[concepts/multi-agent]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/messages]]"
updated: 2026-06-14
---

# メッセージ

**メッセージ**は、受信から返信までの処理パイプラインを束ねる概念――ルーティング/バインディング → セッションキー解決 → キューイング → エージェント実行（ストリーミング＋ツール）→ 送信（チャネル上限＋チャンク化）。各論（[[concepts/queue]]・[[concepts/streaming]]・[[concepts/retry]]）を 1 本の流れに配線する「地図」。

## なぜ重要か

OpenClaw は多数のチャネルを単一の [[components/gateway]] で扱うため、**受信の重複排除・デバウンス・本文の正規化・サイレント返信**といった「チャネルをまたいで一貫させるべき挙動」をこの層に集約している。とくに `BodyForAgent`（モデル向け主テキスト）と `CommandBody`（コマンド解析用の生テキスト）の分離、非ダイレクトチャットでの送信者ラベル付与、`NO_REPLY` のサイレント処理が、マルチユーザー環境での正しさと安全性を支える。

## 押さえる点

- **重複排除**：再接続時の再配信で別実行を起こさない短命キャッシュ。**デバウンス**：同一送信者の連続短文を `messages.inbound` で 1 ターンに結合（メディアは即時、制御コマンドは迂回）。
- **サイレント返信**：`NO_REPLY` はダイレクトで可視フォールバックに書き換え、グループ/内部では許可。内部ランナー障害もグループでは定型文を出さない。
- 設定の置き場：`messages.*`（プレフィックス/キュー/グループ）・`agents.defaults.*`（ストリーミング）・`channels.<ch>.*`（上限/切替）。
- 送受信ライフサイクルの将来設計は [[sources/concepts/message-lifecycle-refactor]]。

詳細・パイプライン図・本文分離は [[sources/concepts/messages]]。

受信メディア（画像/音声/動画）は返信パイプラインの前に [[concepts/media-understanding]] が要約し、ボイスメモの文字起こしはグループのメンション判定にも使われる。

## 関連

- [[concepts/queue]] / [[concepts/streaming]] / [[concepts/retry]] / [[concepts/progress-drafts]]
- [[concepts/session]] / [[concepts/multi-agent]] / [[concepts/media-understanding]] / [[concepts/voice]]
- [[concepts/slash-commands]] — `/...` コマンド/ディレクティブの解析（受信処理の一段）
