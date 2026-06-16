---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/troubleshooting
source_path: raw/docs/nodes/troubleshooting.md
doc_section: nodes
title: "Node のトラブルシューティング"
ingested: 2026-06-14
tags: [node, troubleshooting, permissions, exec-approvals, error-codes]
related:
  - "[[components/node]]"
  - "[[concepts/pairing]]"
  - "[[concepts/diagnostics]]"
---

# Node のトラブルシューティング（解説）

> 原典: `raw/docs/nodes/troubleshooting.md` ・ https://docs.openclaw.ai/ja-JP/nodes/troubleshooting

## 一言まとめ

「ノードは見えているのにノードツールが失敗する」ときの診断手順書。**3 つの別ゲート**（デバイスペアリング / Gateway ノードコマンドポリシー / Exec 承認）のどこで止まっているかを切り分ける。

## 位置づけ

[[components/node]] の運用補助で、[[concepts/diagnostics]]（health/doctor/logs）のノード版。ペアリングは [[concepts/pairing]]、exec 承認は [[sources/gateway/sandboxing]] 周辺。

## 仕組み・ふるまい（切り分け）

1. 基本：`openclaw status` / `gateway status` / `logs --follow` / `doctor` / `channels status --probe`。
2. ノード固有：`openclaw nodes status` / `nodes describe --node <id>` / `approvals get --node <id>`。
3. **3 ゲートの区別**：①デバイスペアリング（Gateway に接続できるか）→ ②ノードコマンドポリシー（`gateway.nodes.allowCommands`/`denyCommands`＋プラットフォーム既定でコマンド ID が許可されているか）→ ③Exec 承認（特定シェルコマンドをローカル実行できるか、`~/.openclaw/exec-approvals.json`）。

## 設定・使い方の要点

- 権限マトリクス：`camera.*`→カメラ権限、`screen.record`→画面収録、`location.get`→位置情報、`system.run`→Exec 承認。`canvas.*`/`camera.*`/`screen.*` は iOS/Android フォアグラウンド必須。
- 高速リカバリ：再ペアリング → アプリをフォアグラウンドに → OS 権限再付与 → Exec 承認ポリシー調整。

## 注意点・落とし穴

- ⚠️ 主要エラーコード：`NODE_BACKGROUND_UNAVAILABLE`（背景）/`CAMERA_DISABLED`/`*_PERMISSION_REQUIRED`/`LOCATION_*`/`SYSTEM_RUN_DENIED: approval required|allowlist miss`。
- **ペアリングは ID/信頼ゲートで、コマンドごとの承認面ではない**。`system.run` 拒否はペアリングでなく Exec 承認の問題。承認後に command/cwd を編集すると `systemRunPlan` 不一致で拒否。

## 用語と略称

- **Exec 承認（exec approvals）** = ノードでのシェル実行を許可する仕組み
- **allowlist miss** = 許可リストに無いコマンドのブロック
- **TCC** = macOS の権限管理（画面収録等）

## 関連ページ

- [[components/node]] / [[concepts/pairing]] / [[concepts/diagnostics]]
- [[sources/nodes/nodes]] / [[sources/gateway/sandboxing]] / [[sources/gateway/troubleshooting]]
