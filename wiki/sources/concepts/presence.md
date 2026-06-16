---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/presence
source_path: raw/docs/concepts/presence.md
doc_section: concepts
title: "プレゼンス"
ingested: 2026-06-14
tags: [presence, gateway, websocket, instances, ttl]
related:
  - "[[concepts/presence]]"
  - "[[components/gateway]]"
  - "[[concepts/architecture]]"
---

# プレゼンス（解説）

> 原典: `raw/docs/concepts/presence.md` ・ https://docs.openclaw.ai/ja-JP/concepts/presence

## 一言まとめ

プレゼンスは、**Gateway 自体と、それに接続しているクライアント**（mac アプリ・WebChat・CLI・Node）を軽量・ベストエフォートに表示するビュー。主に macOS アプリの **Instances** タブでオペレーターが状況を素早く把握するために使われる。

## 位置づけ

[[components/gateway]] が持つ「いま誰がつながっているか」の一時的なビューで、[[concepts/architecture]] の WS 接続トポロジーを運用面から映したもの。プレゼンスエントリは `connect` ハンドシェイクや `system-event` ビーコン、Node 接続から生成・マージされる。

## 仕組み・ふるまい

### フィールド

`instanceId`（安定したクライアント ID、強く推奨）・`host`・`ip`・`version`・`deviceFamily`/`modelIdentifier`・`mode`（`ui`/`webchat`/`cli`/`backend`/`node` 等）・`lastInputSeconds`・`reason`（`self`/`connect`/`node-connected`/`periodic`）・`ts`。

### 生成元（マージされる）

1. **Gateway の self エントリ**：起動時に常に登録（クライアント接続前でも UI にホストが出る）。
2. **WS 接続**：`connect` 成功時にプレゼンスをアップサート。ただし **`mode === "cli"` の単発コマンドはエントリ化しない**（Instances の肥大回避）。
3. **system-event ビーコン**：mac アプリが host/ip/`lastInputSeconds` を定期報告。
4. **Node 接続（`role: node`）**：他の WS クライアントと同様にアップサート。

### マージ・重複排除と寿命

- エントリは**プレゼンスキー**でキー付けされ、最適なキーは再起動後も維持される安定 `instanceId`（大文字小文字を区別しない）。安定 ID 無しで再接続すると**重複行**になりうる。
- **TTL 5 分**（古いエントリは削除）、**最大 200 エントリ**（古いものから削除）。意図的に一時的。

## 設定・使い方の要点

- 生のリストは Gateway に `system-presence` を呼ぶ。macOS Instances タブは最終更新からの経過で Active/Idle/Stale を表示。
- リモート/トンネル：SSH ポートフォワード経由だとリモートアドレスが `127.0.0.1` に見えることがあり、**ループバックのリモートアドレスは無視**してクライアント報告の IP を優先する。

## 注意点・落とし穴

- 重複が出るとき：クライアントがハンドシェイクと定期ビーコンで**同じ安定 `instanceId`** を送っているか確認（接続由来エントリに `instanceId` が無ければ重複は想定どおり）。
- プレゼンスはベストエフォートの表示用であり、信頼できる在席管理の真実源ではない（TTL で消える）。

## 用語と略称

- **プレゼンス（presence）** = Gateway と接続クライアントの軽量な状態ビュー
- **instanceId** = 再起動をまたいで安定したクライアント識別子
- **TTL** = Time To Live（エントリの寿命）
- **アップサート（upsert）** = 無ければ作成・あれば更新
- **ループバック** = `127.0.0.1`（自ホストを指すアドレス）

## 関連ページ

- [[concepts/presence]] — 対応する概念ページ
- [[components/gateway]] / [[components/node]] / [[concepts/architecture]]
- [[concepts/streaming]] / [[concepts/heartbeat]]
