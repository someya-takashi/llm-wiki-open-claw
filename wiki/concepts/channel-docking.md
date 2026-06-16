---
type: concept
aliases: [Channel docking, チャネルのドッキング, dock]
tags: [channel-docking, session, identity-links, routing]
related:
  - "[[concepts/session]]"
  - "[[concepts/channel-routing]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/concepts/channel-docking]]"
updated: 2026-06-14
---

# チャネルのドッキング

**チャネルドッキング**は、1 つの [[concepts/session]] の**「今後の返信の届け先」だけ**を別のリンク済みチャネルへ転送する仕組み。会話コンテキスト（トランスクリプト履歴）は同じセッションのまま、`/dock_<channel>`（例 `/dock_discord`）で返信ルートを切り替える。

## なぜ重要か

「Telegram で始めた作業の続きを Discord で受け取りたい」のような**チャネル横断の作業移動**を、新セッションを切らずに実現する。`/new` が履歴を捨てて作り直すのに対し、ドッキングは**履歴を保ったまま出口だけ**を変える（セッションストアの `lastChannel`/`lastTo`/`lastAccountId` を更新）。アカウント作成・新規ボット接続・アクセス権付与・許可リスト迂回・履歴移動・無関係ユーザーへの共有は**一切しない**――純粋に配信ルートだけを変える安全な操作。

## 押さえる点

- **前提**：`session.identityLinks` に送信元送信者とターゲットピアを同じ ID グループとして登録（値はチャネル接頭辞付き：`telegram:123` `discord:456`）。コマンドはこの値で「同一人物」を証明する。
- コマンドはチャネル Plugin（[[components/gateway]] が読み込む）から生成され、`/dock-discord`/`-slack`/`-telegram`/`-mattermost`（アンダースコア版エイリアスあり）。元に戻すのは元チャネルの dock コマンド。

例・変更/不変フィールド・トラブルシューティングは [[sources/concepts/channel-docking]]。

## 関連

- [[concepts/session]] — DM 分離・identityLinks の文脈
- [[concepts/channel-routing]] / [[components/gateway]]
