---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/plugins/oc-path
source_path: raw/docs/plugins/oc-path.md
doc_section: plugins
title: "OC Path Plugin"
ingested: 2026-06-14
tags: [plugin, oc-path, cli, addressing, markdown, jsonc, jsonl]
related:
  - "[[components/plugin-system]]"
  - "[[components/cli]]"
  - "[[concepts/memory]]"
---

# OC Path Plugin（解説）

> 原典: `raw/docs/plugins/oc-path.md` ・ https://docs.openclaw.ai/ja-JP/plugins/oc-path

## 一言まとめ

同梱・オプトインの `oc-path` Plugin が、`oc://` ワークスペースファイルアドレス指定スキーム用の `openclaw path` CLI を追加する。Markdown/JSONC/JSONL の**単一の葉（値）をバイト忠実に読み書き**するための狭い基盤レイヤー。

## 位置づけ

[[components/plugin-system]] のオプトイン Plugin で、CLI サーフェスは [[components/cli]] の `openclaw path`。高レベルなセマンティクス（メモリ書き込み・設定管理）は所有せず、[[concepts/memory]] 等の上位ツールが周辺に構築する土台。

## 仕組み・ふるまい

- `oc://` アドレスはワークスペースファイル内の 1 葉（or 葉のワイルドカード集合）を指す。理解する 3 種：Markdown（`.md`/`.mdx`：frontmatter/セクション/項目/フィールド）、JSONC（`.jsonc`/`.json5`/`.json`：コメント・書式保持）、JSONL（`.jsonl`/`.ndjson`：行レコード）。
- CLI 内で**インプロセス実行**（Gateway 不要・ソケットを開かない）。`onStartup: false`＋`onCommands: ["path"]` で初回 `openclaw path` 実行時に遅延読み込み。

## 設定・使い方の要点

- 有効化：`openclaw plugins enable oc-path`（Gateway 実行中はマニフェスト反映のため再起動）。動詞：`resolve`/`find`/`set`/`validate`/`emit`（`--json`/`--dry-run`）。
- 用途：ローカル自動化（種別ごとのパーサー不要）・エージェントから見える 1 葉のドライラン差分・エディタ統合・バイト安定性診断。

## 注意点・落とし穴

- `set` は墨消し sentinel ガード付き（`__OPENCLAW_REDACTED__` を含む葉は `OC_EMIT_SENTINEL` で拒否、出力は `[REDACTED]` にスクラブ）。パーサー依存は Plugin ローカル（`commander`/`jsonc-parser`/`markdown-it`）でコアに入らない。

## 用語と略称

- **`oc://`** = ワークスペースファイル内の葉を指すアドレス指定スキーム
- **葉（leaf）** = ファイル内の単一の値ノード
- **JSONC / JSONL** = コメント付き JSON / JSON Lines
- **LKG** = Last-Known-Good（最後に正常だった設定。`oc-path` は関与しない）

## 関連ページ

- [[components/plugin-system]] / [[components/cli]] / [[concepts/memory]]
- [[sources/plugins/manage-plugins]]
