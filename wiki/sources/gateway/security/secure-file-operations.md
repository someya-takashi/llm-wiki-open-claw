---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/security/secure-file-operations
source_path: raw/docs/gateway/security/secure-file-operations.md
doc_section: gateway
title: "安全なファイル操作"
ingested: 2026-06-14
tags: [security, fs-safe, file-operations, path-traversal, library-guardrails]
related:
  - "[[concepts/security]]"
  - "[[sources/gateway/security]]"
  - "[[concepts/sandboxing]]"
---

# 安全なファイル操作（解説）

> 原典: `raw/docs/gateway/security/secure-file-operations.md` ・ https://docs.openclaw.ai/ja-JP/gateway/security/secure-file-operations

## 一言まとめ

OpenClaw は、信頼できないパス名を扱う信頼済みコードのために、`@openclaw/fs-safe` ライブラリで**一貫したガードレール**（ルート境界内の読み書き・アトミック置換・アーカイブ展開・一時ワークスペース・シークレット扱い）を提供する。**これはサンドボックスではない**。

## 位置づけ

[[concepts/security]] の「コードレベルのファイル安全」面。実際の隔離はホストの FS 権限・OS ユーザー・コンテナ・[[concepts/sandboxing]]・ツールポリシーが決め、fs-safe はその上の**ライブラリ的多層防御**。

## 仕組み・ふるまい

- **Python ヘルパーは既定オフ**（`OPENCLAW_FS_SAFE_PYTHON_MODE=off`）：オペレーターが明示選択しない限り Python サイドカーを起動しない（環境間の挙動を予測しやすくするため）。`auto`（あれば使う・無ければ Node にフォールバック）/`require`（起動できなければ fail closed）。
- **Python なしでも保護されるもの（Node パス）**：`..` 相対脱出・絶対パス・名前のみ許可場所の区切り拒否、アドホックな `path.resolve().startsWith()` でなく**信頼済みルートハンドル**で解決、シンボリック/ハードリンクパターンの拒否、識別チェック付きオープン、状態/設定の同階層一時ファイルによる**アトミック書き込み**、読み取り/アーカイブ展開のバイト制限、シークレット/状態のプライベートモード。
- **Python が足すもの**：POSIX で永続 Python プロセスが rename/remove/mkdir/stat に **fd 相対**操作を使い、検証と変更の間に親ディレクトリを差し替える**同一 UID の競合（TOCTOU）窓**を狭める。

## 設定・使い方の要点

- ヘルパーがセキュリティ姿勢の一部なら `auto` でなく **`require`**（`auto` は利用不可時に Node のみへ意図的フォールバック）。`OPENCLAW_FS_SAFE_PYTHON` で明示インタプリタ（汎用名 `FS_SAFE_PYTHON_MODE`/`FS_SAFE_PYTHON` も可）。
- **開発ガイダンス**：Plugin はメッセージ/モデル出力/設定由来のパスに生 `fs` でなく `openclaw/plugin-sdk/*` ヘルパーを、コアは `src/infra/*` の fs-safe ラッパーを使う。アーカイブ展開はサイズ/エントリ/リンク/宛先制限付きヘルパーを。シークレットは独自モードチェックを書かずシークレットヘルパーを。

## 注意点・落とし穴

- ⚠️ **fs-safe はサンドボックスではない**。敵対的ローカルユーザーからの分離が要るなら、別 OS ユーザー/ホストの別 Gateway か [[concepts/sandboxing]] を使う（fs-safe だけに頼らない）。
- カバーするのは OpenClaw の通常の脅威モデル（1 信頼境界内で信頼済みコードが信頼できないパス入力を扱う）。

## 用語と略称

- **fs-safe** = OpenClaw の安全なファイル操作ライブラリ（`@openclaw/fs-safe`）
- **TOCTOU** = Time-Of-Check to Time-Of-Use（検査と使用の間の競合攻撃）
- **fd 相対操作** = ファイルディスクリプタ基準の操作（親差し替えに強い）
- **アトミック書き込み** = 一時ファイル→置換で中途半端な状態を残さない書き込み
- **パストラバーサル** = `..` 等で意図外の場所に到達する攻撃

## 関連ページ

- [[concepts/security]] — 対応する概念ページ
- [[sources/gateway/security]] / [[concepts/sandboxing]] / [[concepts/secrets]]
