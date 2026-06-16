---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/config-tools
source_path: raw/docs/gateway/config-tools.md
doc_section: gateway
title: "設定 — ツールとカスタムプロバイダー"
ingested: 2026-06-14
tags: [configuration, tools, tool-policy, custom-providers, base-url]
related:
  - "[[concepts/configuration]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/agent]]"
---

# 設定 — ツールとカスタムプロバイダー（解説）

> 原典: `raw/docs/gateway/config-tools.md` ・ https://docs.openclaw.ai/ja-JP/gateway/config-tools

## 一言まとめ

`tools.*`（エージェントが使えるツールの許可・制限）と、`models.providers.*`（カスタム/セルフホストのモデルプロバイダーとベース URL）の設定リファレンス。

## 位置づけ

[[concepts/configuration]] のツール＆プロバイダー・ドメイン。ツールポリシーは [[concepts/agent]] のツール面と [[concepts/multi-agent]]/[[concepts/delegate-architecture]] の境界づけ（Gateway レベル強制）に直結し、カスタムプロバイダーは [[concepts/model-providers]]/[[concepts/agent-runtimes]] に対応する。

## 仕組み・ふるまい（主なキー群）

### ツール（`tools.*`）

- **`tools.profile`**：ベース許可リスト（例 `coding`/`messaging`）を `allow`/`deny` より前に設定。**`tools.allow` / `tools.deny`**：個別の許可/拒否。**ツールグループ**（まとめて許可/拒否）。
- **`tools.byProvider`** / **`tools.toolsBySender`**：プロバイダー別・送信者別のツール制御。
- **`tools.elevated`**：昇格実行（グローバル・送信者ベース）。**`tools.exec`**：シェル実行（承認/セキュリティ）。**`tools.loopDetection`**：ツールループ検出。
- **`tools.web`**（Web 検索/取得）・**`tools.media`**（画像/音声/動画）・**`tools.agentToAgent`**（既定オフ・要許可リスト → [[concepts/session-tool]]）・**`tools.sessions`** / **`tools.sessions_spawn`**（サブエージェント生成 → [[concepts/session-tool]]）・**`tools.experimental`**（→ [[sources/concepts/experimental-features]]）。
- **`agents.defaults.subagents`**：委任モード（`suggest`/`prefer`）と並行度（→ [[concepts/parallel-specialist-lanes]]）。

### カスタムプロバイダーとベース URL（`models.providers.*`）

- OpenAI 互換等の**セルフホスト/カスタムプロバイダー**を登録：`baseUrl`・`apiKey`（`${VAR}`/SecretRef）・`api`（`openai-completions`/`ollama` 等）・`models: [{ id, name }]`。
- プロバイダーフィールドの詳細と多数の**プロバイダー例**（Cerebras・ローカル等）を収録。

## 設定・使い方の要点

- **最小ツールサーフェスを優先**：ジョブを完了できる最小限に絞る（プロンプト肥大・誤呼び出し・セキュリティの観点。[[sources/concepts/experimental-features]] のローカルモデル リーンモードも同趣旨）。
- **恒久的に狭くする**なら実験フラグでなく安定ノブ（`tools.profile`/`allow`/`deny`、モデルの `compat.supportsTools: false`）を使う。
- カスタムプロバイダーの API キーは `${VAR}` か SecretRef で（[[sources/gateway/configuration]] の環境変数節）。

## 注意点・落とし穴

- グループ/プロバイダー/サンドボックス/エージェント単位のポリシーは、プロファイル段の**後でさらにツールを削れる**。実際に有効なツールは影響セッションで `/tools` を見て確認。
- `tools.agentToAgent` と `sessions_spawn` は既定で慎重（A2A は要オプトイン＋許可リスト）。

## 用語と略称

- **ツールプロファイル** = `allow`/`deny` の前に効くベース許可リスト
- **昇格（elevated）** = 通常より強い実行権限
- **カスタムプロバイダー** = ベース URL を指定して登録する独自/セルフホストのモデル提供元
- **A2A** = Agent-to-Agent（エージェント間メッセージング）
- **base URL** = プロバイダー API のエンドポイント

## 関連ページ

- [[concepts/configuration]] — 設定の全体像
- [[concepts/model-providers]] / [[concepts/agent-runtimes]] / [[concepts/agent]]
- [[concepts/session-tool]] / [[sources/concepts/experimental-features]] / [[sources/gateway/configuration-reference]]
