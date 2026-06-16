---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/gateway/security/audit-checks
source_path: raw/docs/gateway/security/audit-checks.md
doc_section: gateway
title: "セキュリティ監査チェック"
ingested: 2026-06-14
tags: [security, audit, check-id, reference, hardening]
related:
  - "[[concepts/security]]"
  - "[[sources/gateway/security]]"
---

# セキュリティ監査チェック（解説）

> 原典: `raw/docs/gateway/security/audit-checks.md` ・ https://docs.openclaw.ai/ja-JP/gateway/security/audit-checks
>
> ℹ️ `openclaw security audit` が出す **`checkId` の参照カタログ**（多数）。本解説は全 ID を転記せず、カテゴリと代表例で地図を示す。脅威モデルは [[sources/gateway/security]]。

## 一言まとめ

`openclaw security audit` は `checkId` をキーにした構造化検出結果を出す。各 ID には重大度（重大/警告/情報）・重要な理由・主な修正キー・自動修正可否が付く。本ページはその ID 辞書。

## 位置づけ

[[concepts/security]] の「自動チェック」面。[[concepts/diagnostics]] の health/doctor と並ぶが、こちらは**セキュリティ姿勢**の検査に特化。多くは自動修正できないので、修正キーを見て手で直す。

## 仕組み・ふるまい（checkId のカテゴリ）

- **`fs.*`（ファイル権限）**：状態ディレクトリ/設定/auth-profiles/credentials/sessions/log の world/group-writable・readable・symlink・同期フォルダ（iCloud/Dropbox）。多くは**自動修正あり**。
- **`gateway.*`（公開と認証）**：`bind_no_auth`・`loopback_no_auth`・`trusted_proxies_missing`・`http.no_auth`・`tailscale_funnel`（公開＝重大）・`control_ui.allowed_origins_*`・`token_too_short`・`trusted_proxy_*`・`real_ip_fallback_enabled`。
- **`hooks.*`**：`token_reuse_gateway_token`（重大）・`token_too_short`・`request_session_key_*`・`path_root`・`installs_unpinned/integrity/drift`。
- **`sandbox.*`**：`dangerous_bind_mount`・`dangerous_network_mode`（host/container）・`dangerous_seccomp/apparmor_profile`・`browser_container.non_loopback_publish`。
- **`tools.exec.*`**：`security_full_configured`・`host_sandbox_no_sandbox`・`fs_tools_disabled_but_exec_enabled`・`safe_bins_*`・`allowlist_interpreter_without_strict_inline_eval`。
- **`plugins.*` / `skills.*`**：`*.no_allowlist`・`installs_unpinned/integrity/drift`・`code_safety[.entry_escape/scan_failed]`。
- **`security.exposure.*`**：`open_channels_with_exec`・`open_groups_with_elevated`（重大）・`open_groups_with_runtime_or_fs`。**`security.trust_model.multi_user_heuristic`**（パーソナル前提なのにマルチユーザーに見える）。
- **`models.*`**：`legacy`・`weak_tier`・`small_params`（小モデル＋安全でないツール面でインジェクション増）。**`summary.attack_surface`**（集約サマリー）。
- ほか `discovery.mdns_full_mode`・`browser.*`・`logging.redact_off`（自動修正）。

## 設定・使い方の要点

- 実行：`openclaw security audit`（`checkId` 単位の構造化出力）。重大度の高い `fs.config.perms_world_readable`・`gateway.bind_no_auth`・`hooks.token_reuse_gateway_token`・`security.exposure.open_groups_with_elevated` は最優先で潰す。
- 自動修正可（`fs.*` の多くや `logging.redact_off`）は doctor/audit で直せるが、`gateway.*`/`sandbox.*` の多くは手動で設定キーを修正する。

## 注意点・落とし穴

- 一覧は**網羅ではなく高シグナルの抜粋**。ID とメッセージで分岐し、修正キーのパスをたどる。
- 「警告/重大」が重大度可変のもの（公開度や設定の組み合わせで変わる）は、自分のデプロイ文脈で判断する。

## 用語と略称

- **checkId** = 監査結果を識別する安定したキー
- **重大度（severity）** = 重大 / 警告 / 情報
- **自動修正（auto-fix）** = doctor/audit が直せるか
- **attack surface（攻撃面）** = 攻撃され得る入口の総体

## 関連ページ

- [[concepts/security]] — 対応する概念ページ
- [[sources/gateway/security]] — 脅威モデルと強化
- [[concepts/diagnostics]] — health/doctor との関係
