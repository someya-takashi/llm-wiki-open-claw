---
type: concept
aliases: [Presence, プレゼンス]
tags: [presence, gateway, websocket, instances]
related:
  - "[[concepts/architecture]]"
  - "[[concepts/heartbeat]]"
components:
  - "[[components/gateway]]"
  - "[[components/node]]"
sources:
  - "[[sources/concepts/presence]]"
updated: 2026-06-14
---

# プレゼンス

**プレゼンス**は、[[components/gateway]] 自体とそれに接続しているクライアント（mac アプリ・WebChat・CLI・[[components/node]]）を**軽量・ベストエフォートに表示するビュー**。主に macOS アプリの Instances タブで、オペレーターが「いま誰がつながっているか」を素早く把握するために使う。

## なぜ重要か

[[concepts/architecture]] の WS 接続トポロジーを**運用面から可視化**する仕組み。`connect` ハンドシェイク・`system-event` ビーコン・Node 接続から生成・マージされ、安定した `instanceId` をキーに重複排除される。TTL 5 分・最大 200 エントリと**意図的に一時的**なので、信頼できる在席管理の真実源ではなく「状況把握の窓」と捉えるのが正しい。

## 押さえる点

- 生成元：Gateway の self エントリ／WS 接続（ただし単発 `cli` はエントリ化しない）／system-event ビーコン／Node 接続。
- 重複が出るのは安定 `instanceId` がハンドシェイクと定期ビーコンで揃っていないとき。
- リモート（SSH トンネル）ではループバックのリモートアドレス（`127.0.0.1`）を無視してクライアント報告 IP を優先。
- 生リストは Gateway に `system-presence` を呼ぶ。

フィールド一覧・マージ規則・デバッグは [[sources/concepts/presence]]。

## 関連

- [[concepts/architecture]] / [[components/gateway]] / [[components/node]]
- [[concepts/heartbeat]]
