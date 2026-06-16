---
type: concept
aliases: [Secrets, シークレット, SecretRef]
tags: [secrets, secretref, credentials, vault, security]
related:
  - "[[concepts/configuration]]"
  - "[[concepts/authentication]]"
  - "[[concepts/oauth]]"
components:
  - "[[components/gateway]]"
sources:
  - "[[sources/gateway/secrets]]"
  - "[[sources/gateway/secrets-plan-contract]]"
updated: 2026-06-14
---

# シークレット管理（Secrets）

OpenClaw の **SecretRef**（`{ source: "env"|"file"|"exec", provider, id }`）は、API キーやトークンを **`openclaw.json` に平文で書かずに**、環境変数・ファイル・外部コマンド（Vault/1Password/sops 等）から参照する仕組み。平文も使えて、SecretRef は認証情報ごとにオプトイン。

## なぜ重要か

設定ファイルに秘密を直書きしないことは運用セキュリティの基本だが、「解決のタイミング」を誤るとプロバイダー障害がホットパスに波及する。OpenClaw は秘密値を**起動/リロード時にメモリ内スナップショットへ先行解決**し、ランタイムはそのスナップショットだけを読む（送信ごとに再解決しない）。アクティブな参照が解決できなければ**起動を早期失敗**させ、リロードは**アトミックスワップ**（成功か、最後の正常スナップショット維持）。これにより「秘密を外部化しつつ、実行時は安定」を両立する。

## 押さえる点

- **3 ソース**：`env`（大文字スネークの var 名）／`file`（JSON ポインター、`mode: json|singleValue`、権限チェック）／`exec`（絶対バイナリをシェルなしで実行、JSON プロトコル、`allowSymlinkCommand`/`trustedDirs`）。プロバイダーは `secrets.providers.<name>`。
- **アクティブサーフェスのみ検証**：使われない参照は起動をブロックしない（`SECRETS_REF_IGNORED_INACTIVE_SURFACE`）。
- **オペレーターフロー**：`secrets audit --check` → `secrets configure` → 再監査、適用は `secrets apply --from <plan>`（厳密契約は [[sources/gateway/secrets-plan-contract]]、exec は `--allow-exec` 必須）。
- ⚠️ **一方向の安全ポリシー**：過去の平文値を含むロールバックバックアップは書かない。**OAuth の認証情報には SecretRef を使えない**（静的認証情報専用 → [[concepts/authentication]]）。

詳細（ランタイムモデル・プロバイダー設定・Vault/1Password/sops 例・低下/回復シグナル）は [[sources/gateway/secrets]]。

## 関連

- [[concepts/configuration]] — 設定全体（`${VAR}` 置換と SecretRef）
- [[concepts/authentication]] / [[concepts/oauth]] / [[components/gateway]]
