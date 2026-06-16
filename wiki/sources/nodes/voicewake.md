---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/nodes/voicewake
source_path: raw/docs/nodes/voicewake.md
doc_section: nodes
title: "音声ウェイク"
ingested: 2026-06-14
tags: [voice, wake-word, voicewake, routing, gateway-owned]
related:
  - "[[concepts/voice]]"
  - "[[sources/nodes/talk]]"
  - "[[components/gateway]]"
---

# 音声ウェイク（解説）

> 原典: `raw/docs/nodes/voicewake.md` ・ https://docs.openclaw.ai/ja-JP/nodes/voicewake

## 一言まとめ

ウェイクワード（"openclaw"/"claude"/"computer" 等）を **Gateway が所有する単一のグローバルリスト**として扱う仕組み。どの UI からでも編集でき、Gateway が永続化して全クライアント/ノードにブロードキャストする。

## 位置づけ

[[concepts/voice]] の起動トリガー面。検出した発話は [[sources/nodes/talk]] のループへ繋がる。設定の単一情報源を [[components/gateway]] に置く点は、[[concepts/pairing]] と同じ「Gateway を信頼できる情報源にする」集約思想。

## 仕組み・ふるまい

- **ノードごとのカスタムウェイクワードは無い**（グローバル 1 リスト）。macOS/iOS はローカルの Voice Wake 有効/無効トグルを保持、Android は現状オフ（手動マイク）。
- ストレージ：Gateway の `~/.openclaw/settings/voicewake.json`（`{ triggers, updatedAtMs }`）。
- プロトコル：`voicewake.get`/`set`（トリガー正規化・数/長さ上限）、`voicewake.routing.get`/`set`（トリガー→ターゲット）、イベント `voicewake.changed`/`voicewake.routing.changed`（全 WS クライアント＋接続ノードへ）。
- **ルーティング**：`VoiceWakeRoutingConfig` の各ルートは `{mode:"current"}`／`{agentId}`／`{sessionKey}` のいずれか 1 つ。

## 設定・使い方の要点

- どの UI で編集しても `voicewake.set` を呼び、ブロードキャストで全クライアントが同期。ノード接続時に初期状態がプッシュされる。

## 注意点・落とし穴

- 空リストは既定にフォールバック。Android は Voice Wake 無効（音声タブの手動マイク）。

## 用語と略称

- **ウェイクワード（wake word）** = 音声待受を起動する合言葉
- **ルーティング（routing）** = どのトリガーをどのエージェント/セッションへ向けるか
- **`sessionKey`** = `agent:channel:peer` 形式のセッション識別子

## 関連ページ

- [[concepts/voice]] / [[sources/nodes/talk]] / [[sources/nodes/audio]]
- [[components/gateway]]
