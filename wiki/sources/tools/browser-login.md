---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/browser-login
source_path: raw/docs/tools/browser-login.md
doc_section: tools
title: "ブラウザーログイン"
ingested: 2026-06-16
tags: [browser, login, profile, twitter, sandbox, bot-detection]
related:
  - "[[components/browser]]"
  - "[[concepts/sandboxing]]"
  - "[[concepts/security]]"
---

# ブラウザーログイン（解説）

> 原典: `raw/docs/tools/browser-login.md` ・ https://docs.openclaw.ai/ja-JP/tools/browser-login

## 一言まとめ

ログインの要るサイト（X/Twitter など）を自動化するときの**ログイン方針**――結論は「**モデルに認証情報を渡さず、ホストブラウザーで自分で手動サインインする**」。自動ログインはボット対策を誘発しアカウントロックを招きやすいため。

## 位置づけ

[[components/browser]] の運用ガイドのうち「認証をどう通すか」の章。どのプロファイル（`openclaw` / `user`）を使うか、[[concepts/sandboxing]] が絡むとどう変わるか、を扱う。

## 仕組み・ふるまい（プロファイルの選び方）

- OpenClaw が操作するのは専用の **`openclaw` プロファイル**（オレンジ色 UI、普段使いとは別）。
- **既定は隔離された `openclaw`** を使うべき。既存のログイン済みセッションが重要で、かつユーザーが目の前にいて接続プロンプトを承認できる時のみ `profile="user"`。
- ユーザーブラウザーのプロファイルが複数ある時は推測せず `--browser-profile <name>` で明示。
- アクセス手順：①エージェントにブラウザーを開かせ自分でログイン、または②CLI で `openclaw browser start` → `openclaw browser open https://x.com` を実行して手動サインイン。

## 設定・使い方の要点（サンドボックス × ホスト制御）

[[concepts/sandboxing]] が有効だと、ブラウザーツールの既定は**サンドボックス側**になる。サンドボックス化されたセッションは**ボット検出を誘発しやすい**ため、X/Twitter のような厳格なサイトでは**ホストブラウザー**を優先する。エージェントにホスト制御を許すには：

```json5
{ agents: { defaults: { sandbox: {
  mode: "non-main",
  browser: { allowHostControl: true },
} } } }
```

`sandbox.browser.allowHostControl: true` を立てるとエージェントの `browser` ツール呼び出しがホストを対象にできる（CLI 呼び出しは常にホストブラウザーに対して走る）。あるいは投稿するエージェントのサンドボックス化自体を無効にする。X/Twitter は読み取り・投稿ともホストブラウザー（手動ログイン）が推奨。

## 注意点・落とし穴

- ⚠️ **認証情報をモデルに渡さない**。自動ログイン＝ボット対策誘発＝ロックのリスク。手動ログインが推奨理由。
- ⚠️ `openclaw` プロファイルにはサインイン済みセッションが残る＝機密（[[concepts/security]]）。
- `user`（existing-session）は**ホスト専用**。Linux サーバー/コンテナ/リモートではリモート CDP プロファイルを使う（[[sources/tools/browser-linux-troubleshooting]]）。

## 用語と略称

- **ホストブラウザー** = Gateway と同じマシン上で動く実ブラウザー（サンドボックス内ではない）。
- **`openclaw` / `user` プロファイル** = エージェント専用隔離 / 実サインイン済みブラウザー。
- **ボット対策（bot detection）** = 自動操作を検知してブロックするサイト側の仕組み。

## 関連ページ

- [[components/browser]] / [[concepts/sandboxing]] / [[concepts/security]]
- [[sources/tools/browser]] / [[sources/tools/browser-control]]
- [[sources/tools/browser-linux-troubleshooting]] / [[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]]
