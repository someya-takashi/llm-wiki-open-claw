---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/broadcast-groups
source_path: raw/docs/channels/broadcast-groups.md
doc_section: channels
title: "ブロードキャストグループ"
ingested: 2026-06-14
tags: [broadcast-groups, multi-agent, parallel, sequential, session-isolation, experimental]
related:
  - "[[concepts/groups]]"
  - "[[concepts/multi-agent]]"
  - "[[concepts/channel-routing]]"
---

# ブロードキャストグループ（解説）

> 原典: `raw/docs/channels/broadcast-groups.md` ・ https://docs.openclaw.ai/ja-JP/channels/broadcast-groups
>
> ℹ️ 実験的（2026.1.9 追加）。

## 一言まとめ

1 つのグループメッセージに**複数のエージェントが応答**する仕組み。並列（parallel）/逐次（sequential）の処理戦略を持ち、各エージェントは分離されたセッションで動く。

## 位置づけ

[[concepts/groups]] のブロードキャスト面で、[[concepts/multi-agent]] を「同じグループに複数の頭脳」へ拡張したもの。ルーティングは [[concepts/channel-routing]]。

## 仕組み・ふるまい

- **メッセージフロー**：1 グループメッセージ → 設定された複数エージェントへファンアウト。
- **処理戦略**：`parallel`（既定、同時応答）/`sequential`（順番に、前の出力を次が見られる）。
- **セッション分離**：各エージェントは独自セッションで、それぞれ別のコンテキスト（例 Alfred/Bärbel が別人格で応答）。

## 設定・使い方の要点

- `channels.<provider>.groups.<id>` 配下のブロードキャスト設定（エージェント一覧・処理戦略）。完全な設定スキーマは原典の API リファレンス。
- ユースケース：複数の専門エージェントに同じ質問を投げて見比べる、役割分担した応答。

## 注意点・落とし穴

- ⚠️ **実験的**。プロバイダー/ルーティングの互換性に制約あり。複数エージェントが同時投稿するためノイズになりやすい——用途を絞る。

## 用語と略称

- **ブロードキャスト（broadcast）** = 1 入力を複数エージェントに配る
- **parallel / sequential** = 同時 / 順番の処理戦略
- **セッション分離** = エージェントごとに別の文脈で実行
- **ファンアウト（fan-out）** = 1 つを複数へ展開すること

## 関連ページ

- [[concepts/groups]] / [[concepts/multi-agent]] / [[concepts/channel-routing]]
- [[sources/channels/groups]] / [[concepts/parallel-specialist-lanes]]
