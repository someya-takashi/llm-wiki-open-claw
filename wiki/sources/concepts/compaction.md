---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/concepts/compaction
source_path: raw/docs/concepts/compaction.md
doc_section: concepts
title: "Compaction"
ingested: 2026-06-14
tags: [compaction, context, summary, overflow, memory-flush]
related:
  - "[[concepts/compaction]]"
  - "[[concepts/context]]"
  - "[[concepts/session-pruning]]"
---

# Compaction（解説）

> 原典: `raw/docs/concepts/compaction.md` ・ https://docs.openclaw.ai/ja-JP/concepts/compaction

## 一言まとめ

**Compaction（コンパクション, 圧縮）**は、会話がコンテキストウィンドウの上限に近づいたときに古いメッセージを**要約へまとめ**、チャットを継続可能にする仕組み。要約はトランスクリプトに保存され、直近メッセージはそのまま残る（完全な履歴はディスクに残り、変わるのは次ターンでモデルに見える内容だけ）。

## 位置づけ

[[concepts/context]] をウィンドウ内に収める二大機構の片方（もう片方が [[concepts/session-pruning]]）。[[concepts/agent-loop]] の途中で自動発火し、その実装は [[concepts/context-engine]] が握る（legacy は組み込み要約に委譲）。Compaction の前に [[concepts/memory]] へ重要メモを保存させる「メモリフラッシュ」が走るのが重要な連携点。

## 仕組み・ふるまい

1. 古い会話ターンがコンパクトな要約エントリにまとめられる。
2. 要約はセッションのトランスクリプトに保存される。
3. 直近メッセージはそのまま保持される。

履歴を Compaction チャンクに分割する際、アシスタントのツール呼び出しと対応する `toolResult` は**ペアのまま**保たれる（分割点がツールブロック内なら境界を移動）。

### 自動 Compaction

既定で有効。コンテキスト上限に近づいたとき、またはプロバイダーがオーバーフローエラー（`request_too_large`、`context length exceeded`、`input is too long for the model` 等のシグネチャ）を返したとき（その場合は Compaction して再試行）に走る。`/status` の `🧹 Compactions: <count>` やログの `embedded run auto-compaction start/complete` で確認できる。

### 手動 Compaction

`/compact` で強制でき、`/compact Focus on the API design decisions` のように要約を誘導できる。`keepRecentTokens` があればその切断点を尊重し、無ければハードチェックポイントとして新要約のみから継続する。

## 設定・使い方の要点

`agents.defaults.compaction` 配下：

- **`model`**: 既定はプライマリモデル。要約を高性能/専用モデル（例 `ollama/llama3.1:8b`）へ委任できる。明示上書きは厳密適用でフォールバックチェーンを継承しない。
- **`identifierPolicy`**: 要約は既定で不透明な識別子を保持（`strict`）。`off`/`custom`（＋`identifierInstructions`）。
- **`maxActiveTranscriptBytes`**: アクティブ JSONL がこのサイズに達したら実行前にローカル Compaction（`truncateAfterCompaction: true` が必要）。
- **`truncateAfterCompaction`**: 既存トランスクリプトをその場で書き換えず、要約＋保持状態＋未要約の末尾から**新しい後続トランスクリプト**を作り、旧 JSONL をアーカイブ済みチェックポイントとして残す。
- **`notifyUser`**: 既定は静かに実行。開始/完了の短いステータスを出すには `true`。
- **`memoryFlush.model`**: メモリフラッシュ（Compaction 前のサイレント保存ターン）をローカルモデルで回す上書き。
- **プラグ可能プロバイダー**: Plugin が `registerCompactionProvider()` で要約を委任できる（`compaction.provider` を設定、自動的に `mode: "safeguard"`。失敗時は組み込み LLM 要約にフォールバック）。

## 注意点・落とし穴

- **Compaction vs プルーニング**：Compaction は会話全体を**要約して保存**、[[concepts/session-pruning]] はツール結果を**メモリ内のみトリム**（保存しない）。両者は補完的。
- Compaction が頻繁すぎる→ウィンドウが小さい/ツール出力が大きい。プルーニング有効化を検討。
- Compaction 後にコンテキストが古い→`/compact Focus on <topic>` で誘導、またはメモリフラッシュを使う。クリーンな状態が要るなら `/new`（Compaction せず新セッション）。

## 用語と略称

- **Compaction（コンパクション）** = 古い会話を要約してウィンドウを空ける処理
- **メモリフラッシュ** = Compaction 前に重要メモを [[concepts/memory]] へ保存させるサイレントターン
- **オーバーフロー** = コンテキストウィンドウ超過（プロバイダーエラーで検出）
- **後続トランスクリプト** = 要約から作り直した新しいアクティブ JSONL

## 関連ページ

- [[concepts/compaction]] — 対応する概念ページ
- [[concepts/context]] / [[concepts/session-pruning]] / [[concepts/context-engine]]
- [[concepts/memory]] / [[concepts/agent-loop]]
