---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/secrets
source_path: raw/docs/gateway/secrets.md
doc_section: gateway
title: "シークレット管理"
ingested: 2026-06-14
tags: [secrets, secretref, env, file, exec, vault, snapshot]
related:
  - "[[concepts/secrets]]"
  - "[[concepts/configuration]]"
  - "[[concepts/authentication]]"
---

# シークレット管理（解説）

> 原典: `raw/docs/gateway/secrets.md` ・ https://docs.openclaw.ai/ja-JP/gateway/secrets

## 一言まとめ

API キー等の認証情報を設定に**平文で書かずに済ませる**ための **SecretRef**（`{ source: "env"|"file"|"exec", provider, id }`）の仕組み。秘密値は起動/リロード時に**メモリ内スナップショットへ先行解決**され、ランタイムはそのスナップショットだけを読む。

## 位置づけ

[[concepts/secrets]] の本体。[[concepts/configuration]] のフィールド値（API キー・トークン等）に SecretRef を差し込む形で使い、[[concepts/authentication]]（モデル認証・接続認証）の認証情報を外部化する。平文も引き続き使えて、SecretRef は**認証情報ごとにオプトイン**。

## 仕組み・ふるまい

### ランタイムモデル（スナップショット）

- 解決は**アクティベーション中に先行実行**（リクエストパスで遅延解決しない）。アクティブな SecretRef を解決できなければ起動が早期失敗。リロードは**アトミックスワップ**（完全成功か、直近の正常スナップショット維持）。
- ランタイム/アウトバウンド配信は**アクティブなスナップショットだけ**を読む（送信ごとに再解決しない）→ シークレットプロバイダーの停止がホットパスに波及しない。
- **アクティブサーフェスのみ検証**：無効チャネル・未選択の Web 検索プロバイダーキー・非アクティブな sandbox SSH 認証材料などの未解決参照は起動をブロックせず、`SECRETS_REF_IGNORED_INACTIVE_SURFACE` 診断のみ。

### 3 つのソース

- **env**：`{ source: "env", id: "OPENAI_API_KEY" }`（`id` は大文字スネーク）。
- **file**：`{ source: "file", provider: "filemain", id: "/providers/openai/apiKey" }`（`id` は絶対 JSON ポインター、`mode: "json"|"singleValue"`、所有者/権限チェック）。
- **exec**：`{ source: "exec", provider: "vault", id: "providers/openai/apiKey" }`（絶対バイナリをシェルなしで実行、stdin/stdout の JSON プロトコル、`allowSymlinkCommand`/`trustedDirs`/タイムアウト/env 許可リスト）。

### プロバイダー設定

`secrets.providers.<name>`（`source`＋`path`/`command`/`args`/`passEnv` 等）と `secrets.defaults`（env/file/exec の既定プロバイダー）。exec 統合例：**1Password CLI**（`op read`）・**HashiCorp Vault**（`vault kv get`）・**sops**。MCP サーバー env vars と sandbox SSH 認証材料も SecretRef 対応。

## 設定・使い方の要点

- **オペレーターフロー**：`openclaw secrets audit --check`（平文・未解決 ref・シャドーイング・レガシー残留を検出）→ `openclaw secrets configure`（対話で providers 設定＋SecretRef 化＋プリフライト＋適用）→ 再監査。`secrets apply --from <plan>`（保存計画を適用、`--dry-run`/`--allow-exec`、厳密契約は [[sources/gateway/secrets-plan-contract]]）。
- **アクティベーショントリガー**：起動・設定リロード（hot/restart）・`secrets.reload`・設定書き込み RPC のプリフライト。
- 優先順位：平文と ref が両方ある場合は **ref が優先**（`SECRETS_REF_OVERRIDES_PLAINTEXT` 警告）。`auth-profiles.json` の認証情報が `openclaw.json` の ref より優先される場合は `REF_SHADOWED` 監査検出。

## 注意点・落とし穴

- ⚠️ **一方向の安全ポリシー**：OpenClaw は**過去の平文値を含むロールバックバックアップを書かない**。書き込み前にプリフライト成功＋コミット前にランタイム検証。
- **OAuth ガード**：`type: "oauth"`/`mode: "oauth"` のプロファイルに SecretRef を入れると起動/リロードでハードエラー（ランタイム発行・ローテーション・OAuth 更新用は読み取り専用 SecretRef から意図的に除外）。
- 監査は既定で **exec 解決をスキップ**（副作用回避）。exec を検証/適用するなら `--allow-exec` を dry-run と書き込みの両方で。
- 低下状態：正常後のリロード失敗で `SECRETS_RELOADER_DEGRADED`（最後の正常スナップショット維持）、回復で `SECRETS_RELOADER_RECOVERED`。

## 用語と略称

- **SecretRef** = 秘密値を env/file/exec から参照するオブジェクト
- **スナップショット** = 起動/リロード時に解決済みの秘密値を保持するメモリ内ビュー
- **アクティベーション** = SecretRef を解決してスナップショットを差し替える処理
- **JSON ポインター** = `/a/b` 形式で JSON 内の位置を指す記法（RFC6901）
- **シャドーイング（REF_SHADOWED）** = 別ストアの値が ref を上書きしている状態

## 関連ページ

- [[concepts/secrets]] — 対応する概念ページ
- [[sources/gateway/secrets-plan-contract]] — `secrets apply` の厳密契約
- [[concepts/configuration]] / [[concepts/authentication]] / [[sources/gateway/authentication]]
