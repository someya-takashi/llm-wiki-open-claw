---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/channel-docking
source_path: raw/docs/concepts/channel-docking.md
doc_section: concepts
title: "チャネルのドッキング"
ingested: 2026-06-14
tags: [channel-docking, session, identity-links, routing, slash-commands]
related:
  - "[[concepts/channel-docking]]"
  - "[[concepts/session]]"
  - "[[components/gateway]]"
---

# チャネルのドッキング（解説）

> 原典: `raw/docs/concepts/channel-docking.md` ・ https://docs.openclaw.ai/ja-JP/concepts/channel-docking

## 一言まとめ

チャネルドッキングは、**1 つのセッションの「今後の返信の届け先」だけを別のリンク済みチャネルへ転送**する仕組み。会話コンテキスト（トランスクリプト履歴）は同じセッションのまま保ち、`/dock_<channel>` コマンドで返信ルートを切り替える。

## 位置づけ

[[concepts/session]] のライフサイクルを壊さずに配信先を移すための機能。動作には `session.identityLinks`（同一人物の複数チャネル ID を束ねる設定）が前提で、コマンドは [[components/gateway]] が読み込んだチャネル Plugin から生成される。新セッションを作る `/new` とは違い、**履歴を保ったまま出口だけ変える**点が肝。

## 仕組み・ふるまい

- 例：Alice が `session.identityLinks: { alice: ["telegram:123", "discord:456"] }` を持つとき、Telegram から `/dock_discord` を送ると、以後の返信が Discord `456` に届くようになる（セッションは再作成されない）。
- **変更されるもの**：アクティブセッションの配信フィールド `lastChannel`（例 `discord`）・`lastTo`（例 `456`）・`lastAccountId`。これらはセッションストアに永続化され後続返信に使われる。
- **変更されないもの**：チャネルアカウント作成、新規ボット接続、アクセス権付与、許可リスト/DM ポリシーの迂回、トランスクリプト履歴の移動、無関係ユーザーへの共有――いずれも行わない。**配信ルートだけ**を変える。
- コマンドはバンドル済みで、ターゲットごとに `/dock-discord`（エイリアス `/dock_discord`）・`/dock-mattermost`・`/dock-slack`・`/dock-telegram`。アンダースコア版は Telegram 等のネイティブコマンド面で便利。

## 設定・使い方の要点

- **必須**：`session.identityLinks` に、送信元の送信者とターゲットピアを**同じ ID グループ**として登録する（値はチャネル接頭辞付きピア ID：`telegram:123` `discord:456` `slack:U123`）。正規キー（`alice`）は単なるグループ名で、コマンドは接頭辞付き値で「同一人物」を証明する。
- 元に戻すにはリンク済み送信者から元チャネルの `/dock_telegram` 等を送る。

## 注意点・落とし穴

- 「送信者はリンクされていない」表示 → 送信者とターゲットの両方を同じ `identityLinks` グループに入れる。
- 「アクティブなセッションが無い」表示 → 既存の直接チャットセッションからドックする（新ルートを永続化できるアクティブエントリが要る）。
- 「返信がまだ古いチャネルへ行く」→ コマンド成功とターゲットピア ID の一致を確認。別セッションは別ルートのままのことがある。

## 用語と略称

- **ドッキング** = セッションの返信配信先を別のリンク済みチャネルへ転送すること
- **identityLinks** = 同一人物の複数チャネル ID を束ねる設定
- **ピア ID** = チャネル接頭辞付きの相手識別子（`telegram:123` 等）

## 関連ページ

- [[concepts/channel-docking]] — 対応する概念ページ
- [[concepts/session]] — DM 分離・identityLinks の文脈
- [[components/gateway]] — Dock コマンドを生成するチャネル Plugin の宿主
- [[concepts/channel-routing]]
