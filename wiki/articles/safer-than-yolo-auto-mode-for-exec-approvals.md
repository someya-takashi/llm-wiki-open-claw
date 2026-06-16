---
type: article
source_kind: blog
source_url: https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
source_path: raw/articles/Safer Than YOLO_ Auto Mode for Exec Approvals.md
lang: en
author: [Vincent Koc, Jesse Merhi, Josh Avant]
published: 2026-05-31
summarized: 2026-06-15
title: "YOLO より安全：Exec 承認の auto モード"
tags: [exec, approvals, auto-mode, reviewer, security, codex-guardian]
related:
  - "[[concepts/exec]]"
  - "[[concepts/threat-model]]"
translation: "[[articles/translations/safer-than-yolo-auto-mode-for-exec-approvals]]"
---

# YOLO より安全：Exec 承認の auto モード（要約）

> 原典: `raw/articles/Safer Than YOLO_ Auto Mode for Exec Approvals.md` ・ https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
> 全文翻訳（一文一文の忠実訳）は [[articles/translations/safer-than-yolo-auto-mode-for-exec-approvals]]
> ℹ️ OpenClaw 公式ブログの告知記事（二次資料）。著者: Vincent Koc / Jesse Merhi / Josh Avant、公開 2026-05-31。

## 一言まとめ

Exec 承認に、`ask`（厳格・人間優先）と `YOLO`（承認なし）の中間となる**第 3 のモード `auto`（レビュアー優先）**をオプトインで追加する、という告知。決定的に安全なコマンドはそのまま実行し、ポリシーを外れたものはまず**モデルがレビュー**、確信が持てなければ人間にフォールバックする。

## 位置づけ（既存 wiki との接続）

これは [[concepts/exec]]（コマンド実行と承認の面）の**新しい承認モードの告知**で、既存の [[sources/tools/exec-approvals]]（`exec.security`/`exec.ask`/YOLO）に `auto` を足すもの。発想元は OpenAI Codex の Guardian レビュー済み承認（[[sources/plugins/codex-harness]]）で、それをホスト exec 一般に持ち込む。承認をチャネルへ転送する話は [[sources/tools/exec-approvals-advanced]]、信頼できない入力を「データとして扱う」防御は [[concepts/threat-model]] のプロンプトインジェクション対策と整合する。

> ⚠️ **二次資料・新機能の告知**：本記事の `tools.exec.mode: "auto"` 等はブログ時点の情報で、公式 docs（[[sources/tools/exec-approvals]]）にはまだ載っていない可能性がある。詳細は本ページに置き、[[concepts/exec]] 本文（公式 docs ベース）は書き換えない。

## 要点（記事の主張と新規情報）

- **3 モードの位置づけ**：
  - `ask`（Human-first）= 許可リストの取りこぼしは停止して人間を待つ（厳格だがうるさい）
  - `auto`（Reviewer-first・**新規**）= 決定的一致は実行、取りこぼしは OpenClaw ネイティブの auto レビュアー → 人間フォールバック
  - `YOLO`（No prompts）= 承認なしで実行（外部サンドボックス済み環境のみ）
- **auto の処理順**：①許可リスト/safe-bin 一致 → 実行 ②外れたら境界付きレビューパケット（command/argv/cwd/env キー名/host/パーサー解析）を作る ③auto レビュアーは**低リスクの実行を 1 回だけ**許可 ④曖昧・高リスク・解析不能・タイムアウト・モデル不能・レビュアー指示は人間へ ⑤応答できる UI/クライアントが無ければホストのフォールバック。
- **レビュアーモデルはエージェントと別**（新規キー `tools.exec.reviewer.model`）：通常はローカルモデル、承認判断だけ `openai/gpt-5.5` 等のフロンティアに向けられる。
- **ローカル安全設定を上書きしない**：常に ask のホストは ask、deny のホストは deny のまま。
- **承認のチャット転送**：Slack/Telegram/iMessage 等オペレーターが見る場所へ承認プロンプトを送れる。
- **デフォルトは変更しない**：`auto` は公開テスト中のオプトイン。Enterprise 環境に最適と位置づけ。

## 設定（記事より）

```bash
openclaw config set tools.exec.host gateway
openclaw config set tools.exec.mode auto
# レビューに強いモデルを使う場合のみ:
openclaw config set tools.exec.reviewer.model openai/gpt-5.5
```

## なぜ重要か

[[concepts/exec]] は「`exec` は変更可能なシェル面で、承認設計が安全性の要」という領域。`auto` は「厳格な ask の鬱陶しさ」と「YOLO の無防備さ」のジレンマに、**モデルによる中間レビュー＋人間フォールバック**という第 3 の解を与える点が新しい。レビュアーがコマンド文字列等を**信頼できないデータ**として扱い、指示に乗らず人間に委ねる設計は、[[concepts/threat-model]] の「exec 承認バイパス」「プロンプトインジェクション」への直接の対処になっている。

## 関連ページ

- [[articles/translations/safer-than-yolo-auto-mode-for-exec-approvals]] — 全文翻訳
- [[concepts/exec]] / [[sources/tools/exec-approvals]] / [[sources/tools/exec-approvals-advanced]] / [[sources/tools/elevated]]
- [[sources/plugins/codex-harness]]（Codex Guardian） / [[concepts/threat-model]] / [[concepts/security]]
