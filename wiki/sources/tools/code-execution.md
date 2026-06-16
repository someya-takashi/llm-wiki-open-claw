---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/tools/code-execution
source_path: raw/docs/tools/code-execution.md
doc_section: tools
title: "コード実行（code_execution）"
ingested: 2026-06-14
tags: [code-execution, python, xai, provider-assisted, sandboxed-remote]
related:
  - "[[concepts/exec]]"
  - "[[concepts/agent-runtimes]]"
  - "[[components/plugin-system]]"
---

# コード実行（code_execution）（解説）

> 原典: `raw/docs/tools/code-execution.md` ・ https://docs.openclaw.ai/ja-JP/tools/code-execution

## 一言まとめ

`code_execution` は **xAI の Responses API でサンドボックス化されたリモート Python 解析**を実行するツール。同梱 `xai` Plugin が登録し、`x_search` と同じ `https://api.x.ai/v1/responses` にディスパッチする。

## 位置づけ

[[concepts/exec]] の「ランタイム」カテゴリの一つだが、ローカルシェルの `exec` とは別物——**プロバイダー側でホストされる Python インタプリタ**。実装は [[components/plugin-system]] の `xai` Plugin（`tools` コントラクト）、ランタイム選択は [[concepts/agent-runtimes]]。

## 仕組み・ふるまい

- xAI の Responses API にプロンプト＋コードをディスパッチし、リモートのサンドボックスで Python を実行して結果を返す。ローカルホストでは実行しない。

## 設定・使い方の要点

- セットアップは xAI の API キー（`XAI_API_KEY`）。`x_search`（X 投稿検索）と同じエンドポイント/Plugin を共有。

## 注意点・落とし穴

- ⚠️ ローカルの `exec`（[[sources/tools/exec]]）とは認可境界が違う——こちらはプロバイダー側サンドボックスでの実行で、ホストのファイルには触れない。制限・エラーは原典参照。

## 用語と略称

- **code_execution** = プロバイダー側でホストされる Python 実行ツール
- **Responses API** = xAI の応答 API（`/v1/responses`）
- **サンドボックス化リモート実行** = プロバイダー側の隔離環境での実行
- **`x_search`** = X（Twitter）投稿検索ツール（同じ xAI Plugin）

## 関連ページ

- [[concepts/exec]] / [[concepts/agent-runtimes]] / [[components/plugin-system]]
- [[sources/tools/exec]] / [[providers/xai]]（未作成）
