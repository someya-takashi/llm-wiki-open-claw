---
type: concept
aliases: [Groups, グループ, group chat, mention gating]
tags: [groups, group-chat, mention-gate, allowlist, broadcast, access-groups]
related:
  - "[[concepts/channel-routing]]"
  - "[[concepts/messages]]"
  - "[[concepts/security]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/channels/groups]]"
  - "[[sources/channels/broadcast-groups]]"
  - "[[sources/channels/access-groups]]"
  - "[[sources/channels/group-messages]]"
updated: 2026-06-14
---

# グループ（Groups）

グループ（groups, グループチャットでのエージェントの振る舞い）は、OpenClaw が Discord/iMessage/Matrix/MS Teams/Signal/Slack/Telegram/WhatsApp/Zalo の**グループチャットを一貫して扱う**ための仕組み。OpenClaw は「あなた自身のアカウント上で動く」ため、**あなたが参加しているグループ**を認識し、そこで応答できる（別個のボットユーザーは不要）。

## なぜ重要か

DM（1 対 1）と違い、グループは「全員に応答すると鬱陶しい/危険」「誰の発言を信頼するか」が問題になる。だから OpenClaw は既定で**メンションゲート**（呼ばれたときだけ起動）と**許可リスト**でグループの振る舞いを絞る。これは [[concepts/channel-routing]] の下位の振る舞い領域であり、[[concepts/security]] の境界（オープングループ＋高権限の事故を防ぐ）と直結する。

## 4 つの軸

| 軸 | 何を決めるか | ソース |
|---|---|---|
| **メンションゲート** | グループで応答する条件（既定 mention） | [[sources/channels/groups]] |
| **許可リスト/アクセスグループ** | 誰の発言を受け付けるか（`accessGroup:<name>`） | [[sources/channels/access-groups]] |
| **グループごとのツール制限** | グループで使えるツール/サンドボックス | [[sources/channels/groups]]・[[concepts/sandboxing]] |
| **ブロードキャスト** | 1 グループに複数エージェントが応答（実験的） | [[sources/channels/broadcast-groups]] |

## 仕組み（要点）

- **メンションゲート（既定）**：`requireMention` 等で「呼ばれたときだけ」。ボイスメモは preflight 文字起こしでメンション判定（[[sources/nodes/audio]]）。
- **グループセッション**：グループは個人 DM と別セッション（`agent:<id>:<channel>:<groupId>` 系）。コンテキストの可視性/許可リストでサンドボックス化（DM はホスト、グループは隔離など）。
- **アクティベーション（オーナーのみ）**：`/activation mention|always`（[[concepts/slash-commands]]）。
- **ブロードキャストグループ**：1 つのグループメッセージに複数エージェントが parallel/sequential で応答、セッション分離（[[sources/channels/broadcast-groups]]）。

## 既存 wiki とのつながり

グループの受信処理は [[concepts/messages]] のメンションゲート/デバウンス、配信は [[concepts/channel-routing]]。アクセスグループ（`accessGroup:<name>`）は [[concepts/security]] の許可リストを名前付きで再利用する仕組みで、[[sources/gateway/operator-scopes]] とは別レイヤー（送信者認可）。各チャネル固有の挙動は [[channels/whatsapp]] 等。

## 代表ソース

- [[sources/channels/groups]] — クロスチャネルのグループモデル（メンションゲート・許可リスト・ツール制限）
- [[sources/channels/broadcast-groups]] — 複数エージェントの並列/逐次応答（実験的）
- [[sources/channels/access-groups]] — 名前付き送信者グループ（`accessGroup:`）
- [[sources/channels/group-messages]] — WhatsApp 固有のグループ挙動

## 関連ページ

- [[concepts/channel-routing]] / [[concepts/messages]] / [[concepts/security]] / [[concepts/sandboxing]]
- [[concepts/pairing]] / [[concepts/multi-agent]] / [[components/gateway]]
