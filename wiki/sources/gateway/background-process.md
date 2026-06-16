---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/background-process
source_path: raw/docs/gateway/background-process.md
doc_section: gateway
title: "バックグラウンド実行とプロセスツール"
ingested: 2026-06-14
tags: [exec, process, background, pty, tools, shell]
related:
  - "[[concepts/agent]]"
  - "[[concepts/heartbeat]]"
  - "[[sources/gateway/config-tools]]"
---

# バックグラウンド実行とプロセスツール（解説）

> 原典: `raw/docs/gateway/background-process.md` ・ https://docs.openclaw.ai/ja-JP/gateway/background-process

## 一言まとめ

エージェントの **`exec` ツール**（シェルコマンド実行）と **`process` ツール**（その長時間バックグラウンドセッションの管理）。長い処理を**メモリ内に保持**し、後からポーリング・入力送信・停止できる。

## 位置づけ

[[concepts/agent]] のコアツール（read/exec/edit/write）のうち、長時間作業を扱う面。バックグラウンド完了時に Heartbeat を起こす（[[concepts/heartbeat]]）連携があり、ツールポリシー設定は [[sources/gateway/config-tools]]。明示的な将来作業は Cron（automation）に委ねる設計。

## 仕組み・ふるまい

### `exec` ツール

- 主パラメータ：`command`（必須）・`yieldMs`（既定 10000、この遅延後に自動バックグラウンド化）・`background`（即バックグラウンド）・`timeout`（秒、既定 `tools.exec.timeoutSec`、`0` で無効）・`elevated`（サンドボックス外実行）・`pty`（実 TTY）・`workdir`/`env`。
- フォアグラウンドは出力を直接返す。バックグラウンド化すると `status: "running"` ＋`sessionId`＋末尾出力を返し、**出力はポーリング/クリアまでメモリ内に保持**。`process` ツールが不許可なら `exec` は同期実行（`yieldMs`/`background` を無視）。

### `process` ツール（アクション）

`list`（実行中＋完了）/`poll`（新出力をドレイン＋終了ステータス）/`log`（集約出力、`offset`+`limit`）/`write`（stdin、`eof` 可）/`send-keys`（PTY にキー）/`submit`（Enter）/`paste`（リテラル、bracketed paste 可）/`kill`/`clear`/`remove`。**エージェントごとスコープ**（自分が開始したセッションのみ）。`waitingForInput` は書き込み可能な stdin がアイドル閾値超で表示。

## 設定・使い方の要点

- 設定：`tools.exec.backgroundMs`（既定 10000）・`timeoutSec`（既定 1800）・`cleanupMs`（既定 1800000）・`notifyOnExit`（既定 true＝終了時にシステムイベントを入れて Heartbeat 要求）・`notifyOnExitEmptySuccess`（既定 false）。env 上書き：`PI_BASH_YIELD_MS`/`PI_BASH_MAX_OUTPUT_CHARS`/`PI_BASH_JOB_TTL_MS`（1m–3h）等。
- **子プロセスブリッジ**：exec/process 外で長時間子プロセスを生成するときはブリッジヘルパーを付け、終了シグナル転送・リスナーのデタッチで systemd 上の孤立を防ぐ。
- 例：`{tool:"exec", command:"sleep 5 && echo done", yieldMs:1000}` → `{tool:"process", action:"poll", sessionId:"<id>"}`。

## 注意点・落とし穴

- **セッションはメモリ内のみ**（ディスク永続化なし）→ プロセス再起動で失われる。ログがチャット履歴に残るのは `process poll/log` でツール結果が記録されたときだけ。
- ⚠️ **`sleep` ループや反復ポーリングでリマインダー/遅延フォローアップを擬似しない**。将来の作業は Cron を使う（ポーリングはオンデマンドのステータス確認用）。

## 用語と略称

- **exec / process** = コマンド実行 / バックグラウンドセッション管理のツール
- **PTY** = 擬似端末（インタラクティブ CLI に必要）
- **yieldMs / background** = 自動バックグラウンド化までの遅延 / 即バックグラウンド化
- **bracketed paste** = 端末がペーストを 1 塊と認識するモード
- **stdin / EOF** = 標準入力 / 入力終端

## 関連ページ

- [[concepts/agent]] — コアツールの文脈
- [[concepts/heartbeat]] — `notifyOnExit` による起床
- [[sources/gateway/config-tools]] — `tools.exec` 設定
