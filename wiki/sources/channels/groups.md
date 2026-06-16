---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/channels/groups
source_path: raw/docs/channels/groups.md
doc_section: channels
title: "グループ"
ingested: 2026-06-14
tags: [groups, group-chat, mention-gate, allowlist, tool-restrictions, sandbox]
related:
  - "[[concepts/groups]]"
  - "[[concepts/messages]]"
  - "[[concepts/sandboxing]]"
---

# グループ（解説）

> 原典: `raw/docs/channels/groups.md` ・ https://docs.openclaw.ai/ja-JP/channels/groups

## 一言まとめ

Discord/iMessage/Matrix/MS Teams/Signal/Slack/Telegram/WhatsApp/Zalo の**グループチャットを一貫して扱う**クロスチャネルモデル。OpenClaw は自分のアカウント上で動くので、あなたが参加しているグループを認識し、メンションされたときだけ応答する。

## 位置づけ

[[concepts/groups]] の中核ソース。受信処理は [[concepts/messages]]、ツール制限/隔離は [[concepts/sandboxing]]、宛先は [[concepts/channel-routing]]。

## 仕組み・ふるまい

- **メンションゲート（既定）**：グループでは呼ばれたときだけ起動。`/activation mention|always`（オーナーのみ）。
- **コンテキストの可視性と許可リスト**：「個人 DM はホスト、グループはサンドボックス化」「グループには許可リストのフォルダーだけ見せる」等のパターン。
- **グループポリシー**：グループ allowlist（全グループ返信を無効/特定グループのみ/全許可＋メンション必須/オーナーのみトリガー）。
- **グループ/チャンネルのツール制限**：グループでは狭いツールセットに（[[sources/tools/multi-agent-sandbox-tools]]）。
- セッションキーはグループごと（個人 DM と分離）。表示ラベル・コンテキストフィールド。

## 設定・使い方の要点

- `channels.<provider>.groups.<groupId>`（メンション要求・ツール制限・許可リスト）。iMessage/WhatsApp 固有事項あり。WhatsApp はシステムプロンプトにグループ文脈を注入。

## 注意点・落とし穴

- ⚠️ オープングループ＋広いツール/exec は高リスク（[[concepts/security]]・[[concepts/threat-model]]）。グループでは `/reasoning`/`/verbose` を晒さない。

## 用語と略称

- **メンションゲート** = グループで応答する条件（@呼びかけ等）
- **グループ allowlist** = 応答を許すグループ/送信者のリスト
- **アクティベーション** = グループでの起動モード（mention/always）
- **コンテキストの可視性** = グループに見せるファイル/情報の範囲

## 関連ページ

- [[concepts/groups]] — 対応する概念ページ
- [[concepts/messages]] / [[concepts/sandboxing]] / [[concepts/channel-routing]]
- [[sources/channels/access-groups]] / [[sources/channels/broadcast-groups]] / [[sources/channels/group-messages]]
