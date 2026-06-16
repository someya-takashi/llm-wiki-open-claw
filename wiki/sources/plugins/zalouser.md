---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/zalouser
source_path: raw/docs/plugins/zalouser.md
doc_section: plugins
title: "Zalo 個人用 Plugin"
ingested: 2026-06-14
tags: [plugin, channel, zalo, zalouser, messaging, qr-login, unofficial]
related:
  - "[[channels/zalouser]]"
  - "[[components/plugin-system]]"
  - "[[concepts/messages]]"
---

# Zalo 個人用 Plugin（解説）

> 原典: `raw/docs/plugins/zalouser.md` ・ https://docs.openclaw.ai/ja-JP/plugins/zalouser

## 一言まとめ

通常の Zalo（ベトナムのメッセージングアプリ）**個人ユーザーアカウント**を、ネイティブの `zca-js` で自動化する Plugin（QR ログイン＋メッセージング）。非公式自動化のためアカウント停止リスクがある。

## 位置づけ

[[components/plugin-system]] が提供する**チャネル**統合（チャネルページは [[channels/zalouser]]）。受信→返信は [[concepts/messages]] の共通パイプラインに乗る。チャネル id は `zalouser`（個人アカウント自動化を明示。`zalo` は将来の公式 API 用に予約）。

## 仕組み・ふるまい

- [[components/gateway]] プロセス内で実行（外部 `zca`/`openzca` バイナリ不要）。リモート Gateway では Gateway ホストにインストール・設定・再起動。
- エージェントツール `zalouser`：`send`/`image`/`link`/`friends`/`groups`/`me`/`status`（チャネルメッセージは `react` も）。

## 設定・使い方の要点

- インストール：`openclaw plugins install @openclaw/zalouser`（or ローカル）。設定は **`channels.zalouser`**（`plugins.entries.*` ではない）：`enabled`・`dmPolicy: "pairing"` 等。
- CLI：`openclaw channels login --channel zalouser`（QR）・`logout`・`status --probe`・`message send --channel zalouser --target <threadId>`・`directory peers list`。

## 注意点・落とし穴

- ⚠️ **非公式自動化**でアカウント停止/BAN の可能性（自己責任）。チャネル設定は `channels.zalouser` 配下で、Plugin エントリ設定とは別の場所にある点に注意。

## 用語と略称

- **Zalo** = ベトナムの主要メッセージングアプリ
- **zca-js** = Zalo の非公式クライアントライブラリ
- **QR ログイン** = QR コードでアカウントを連携するログイン
- **dmPolicy** = DM の受け入れポリシー（pairing 等）

## 関連ページ

- [[channels/zalouser]] — 対応するチャネルページ
- [[components/plugin-system]] / [[concepts/messages]] / [[components/gateway]]
