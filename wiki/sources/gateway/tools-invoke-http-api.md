---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/tools-invoke-http-api
source_path: raw/docs/gateway/tools-invoke-http-api.md
doc_section: gateway
title: "ツール呼び出し API"
ingested: 2026-06-14
tags: [http-api, tools-invoke, tool-policy, deny-list, rce]
related:
  - "[[concepts/http-api]]"
  - "[[concepts/session-tool]]"
  - "[[concepts/security]]"
---

# ツール呼び出し API（解説）

> 原典: `raw/docs/gateway/tools-invoke-http-api.md` ・ https://docs.openclaw.ai/ja-JP/gateway/tools-invoke-http-api

## 一言まとめ

**単一ツールを直接叩く** `POST /tools/invoke`。`/v1/*` と違い**常時有効**で、Gateway 認証＋ツールポリシーを使う。エージェントの推論を介さずツールを 1 回実行する低レベル面。

## 位置づけ

[[concepts/http-api]] の 3 つ目の面（チャット系ではなくツール系）。実行されるツールは [[concepts/session-tool]] のツール群で、可用性は Gateway エージェントと同じポリシーチェーンでフィルタされる。

## 仕組み・ふるまい

- **本文**：`tool`（必須）・`action`・`args`・`sessionKey`（省略/`"main"` で既定メイン）・`dryRun`（予約・現状無視）。既定最大ペイロード 2MB。
- **ポリシーチェーン**：`tools.profile`/`allow` → `byProvider` → `agents.<id>` → グループ → サブエージェント。許可されないツールは **404**。`x-openclaw-message-channel`/`x-openclaw-account-id` でグループ文脈を解決。
- **レスポンス**：`200 {ok:true,result}` / `400`（入力エラー）/ `401` / `404`（未許可）/ `405` / `429`（レート制限）/ `500`（サニタイズ済みメッセージ）。

## 設定・使い方の要点

- 認証は Gateway 認証モード（token/password→bearer、trusted-proxy、none）。
- ⚠️ **HTTP ハード拒否リスト（既定、セッションポリシーが許可していてもブロック）**：`exec`/`spawn`/`shell`（RCE 面）、`fs_write`/`fs_delete`/`fs_move`/`apply_patch`（任意ファイル変更）、`sessions_spawn`/`sessions_send`、`cron`、`gateway`（HTTP 経由の再設定防止）、`nodes`（ペアリング済みホストの `system.run` に到達しうる）、`whatsapp_login`（対話セットアップ）。`gateway.tools.deny`/`allow` でカスタマイズ。

## 注意点・落とし穴

- ⚠️ **完全なオペレーターアクセス**：共有シークレット bearer は所有者資格情報相当。狭い `x-openclaw-scopes` は無視され、フルのオペレーター既定＋所有者送信ターン扱い。loopback/tailnet/プライベート入口のみ。
- ⚠️ **Exec 承認はオペレーターのガードレールであって、この HTTP 面の認可境界ではない**。ポリシーでツールに到達できれば追加の都度承認は入らない。`exec` に到達できるなら「変更可能なシェル面」とみなす（`write` を拒否してもシェル実行は読み取り専用にならない）。
- 信頼境界の分離が要るなら**別 Gateway（理想は別 OS ユーザー/ホスト）**を立てる。

## 用語と略称

- **RCE** = Remote Code Execution（遠隔コード実行）
- **拒否リスト（deny-list）** = HTTP 経由で禁止するツールの既定集合
- **ポリシーチェーン** = ツール可否を段階的に絞る設定の連なり

## 関連ページ

- [[concepts/http-api]] — 対応する概念ページ
- [[sources/gateway/openai-http-api]] / [[concepts/session-tool]]
- [[concepts/security]] / [[concepts/sandboxing]] / [[components/gateway]]
