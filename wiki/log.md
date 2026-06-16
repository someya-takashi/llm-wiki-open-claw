# Log — OpenClaw LLM Wiki 時系列ログ

append-only。新しいエントリは末尾に追記する。`grep "^## \[" log.md | tail -10` で直近の動きを追える。

## [2026-06-14] schema-update | OpenClaw 向けに初期化

- このリポジトリは 3D Vision 版 LLM Wiki からコピーして作成された。対象ドメインを **OpenClaw**（あらゆる OS で動く AI エージェント用のセルフホスト型 Gateway）へ作り変えた。
- `CLAUDE.md` を全面改訂：領域説明・ディレクトリ構成・命名規約・frontmatter（`source` / `concept` / `component` / `channel` / `provider` / `question`）・§4 の略称例を OpenClaw 用に差し替え。**翻訳工程（translations）を廃止**（原典の公式ドキュメントが既に日本語のため）。
- カテゴリを `concepts/` `components/` `channels/` `providers/` ＋ `sources/` `questions/` に設定。
- 原典（`raw/docs/`）と解説（`wiki/sources/`）は、公式ドキュメントの URL ツリー（`<section>/<slug>`）を **サブフォルダでミラー**する方針に決定。
- skill を改訂：`ingest`（clip 移送＋解説生成、Mermaid 図の描き直し、翻訳ステップ削除）、`query`、`lint`（sources のツリー整合点検を追加）。
- ディレクトリ作成：`raw/docs/` `raw/articles/` `raw/assets/`、`wiki/{sources,concepts,components,channels,providers,questions}/`。
- 初期ファイル作成：`wiki/index.md` `wiki/log.md` `wiki/overview.md`。
- 既存 clip `Clippings/Gateway アーキテクチャ.md`（元: https://docs.openclaw.ai/ja-JP/concepts/architecture）を `raw/docs/concepts/architecture.md` へ移送。**本格 ingest は未実施**（最初の ingest 候補）。

## [2026-06-14] schema-update | 原典の移送を ingest 後に変更

- ingest skill のフローを変更：clip を **ingest の最初に** `raw/docs/` へ移すのではなく、**ingest 完了後（最終ステップ 10）** に `Clippings/<title>.md` → `raw/docs/<section>/<slug>.md` へ移送するようにした。作業中は clip を `Clippings/` から読み、frontmatter の `source_path`・「原典」行には最終置き場のパスを書く。
- 反映ファイル: `.claude/skills/ingest/SKILL.md`（description・ステップ1・新ステップ10）、`CLAUDE.md`（§1 Clippings 注記・§3 ingest 説明）。

## [2026-06-14] ingest | concepts コア4ページ（architecture / agent / agent-runtimes / agent-loop）

- 取り込み: `Clippings/` の 4 clip を一括 ingest
  - https://docs.openclaw.ai/ja-JP/concepts/architecture → `raw/docs/concepts/architecture.md`（init 移送済みと同一の再 clip。上書き）
  - https://docs.openclaw.ai/ja-JP/concepts/agent → `raw/docs/concepts/agent.md`
  - https://docs.openclaw.ai/ja-JP/concepts/agent-runtimes → `raw/docs/concepts/agent-runtimes.md`
  - https://docs.openclaw.ai/ja-JP/concepts/agent-loop → `raw/docs/concepts/agent-loop.md`
- 作成（sources）: [[sources/concepts/architecture]], [[sources/concepts/agent]], [[sources/concepts/agent-runtimes]], [[sources/concepts/agent-loop]]
- 作成（concepts）: [[concepts/architecture]], [[concepts/agent]], [[concepts/agent-runtimes]], [[concepts/agent-loop]]
- 作成（components）: [[components/gateway]], [[components/node]]
- 更新: [[index]], [[overview]]
- メモ:
  - ユーザー確定で、混同しやすい 2 概念は docs の URL スラグをミラー（`concepts/agent` = 1 プロセスの契約 / `concepts/agent-runtimes` = 実行バックエンド層）。両ページと index・overview に「混同注意」の区別ノートを明記。
  - architecture の接続ライフサイクル図（巨大インライン SVG）は保存せず Mermaid sequenceDiagram で描き直し。agent-loop は全体フローを Mermaid flowchart で新規作図。
  - dangling link（未作成）: concepts/{pairing,queue,session,compaction,streaming,multi-agent,model-providers,system-prompt,agent-workspace}, components/cli, providers/{openai,anthropic}。次回以降の ingest 候補。

## [2026-06-14] ingest | concepts コンテキスト/ワークスペース群 6ページ（agent-workspace / soul / system-prompt / context / context-engine / oauth）

- 取り込み: `Clippings/` の 6 clip を一括 ingest
  - https://docs.openclaw.ai/ja-JP/concepts/agent-workspace → `raw/docs/concepts/agent-workspace.md`
  - https://docs.openclaw.ai/ja-JP/concepts/soul → `raw/docs/concepts/soul.md`
  - https://docs.openclaw.ai/ja-JP/concepts/system-prompt → `raw/docs/concepts/system-prompt.md`
  - https://docs.openclaw.ai/ja-JP/concepts/context → `raw/docs/concepts/context.md`
  - https://docs.openclaw.ai/ja-JP/concepts/context-engine → `raw/docs/concepts/context-engine.md`
  - https://docs.openclaw.ai/ja-JP/concepts/oauth → `raw/docs/concepts/oauth.md`
- 作成（sources）: [[sources/concepts/agent-workspace]], [[sources/concepts/soul]], [[sources/concepts/system-prompt]], [[sources/concepts/context]], [[sources/concepts/context-engine]], [[sources/concepts/oauth]]
- 作成（concepts）: [[concepts/agent-workspace]], [[concepts/soul]], [[concepts/system-prompt]], [[concepts/context]], [[concepts/context-engine]], [[concepts/oauth]]
- 更新: [[concepts/agent]]（related に agent-workspace/system-prompt/soul を追加・本文補足）, [[concepts/agent-loop]]（related に system-prompt/context を追加）, [[index]], [[overview]]
- メモ:
  - 前回 ingest の dangling link だった [[concepts/agent-workspace]] と [[concepts/system-prompt]] を解消。
  - 図は無し（context の `/context map` ツリーマップは画像だが本質ではないため文章で代替）。ラスター画像の取り込みも無し。
  - 新規 dangling link（次回候補）: concepts/{memory,session,compaction,session-pruning,sandboxing,model-providers,model-failover,multi-agent,pairing}, providers/{openai,anthropic}, gateway/{configuration,secrets,authentication,sandboxing}, automation/hooks, tools/slash-commands, plugins/architecture。

## [2026-06-14] ingest | concepts セッション群＋QA＋start/bootstrapping 8ページ

- 取り込み: `Clippings/` の 8 clip を一括 ingest
  - https://docs.openclaw.ai/ja-JP/concepts/session → `raw/docs/concepts/session.md`
  - https://docs.openclaw.ai/ja-JP/concepts/session-pruning → `raw/docs/concepts/session-pruning.md`
  - https://docs.openclaw.ai/ja-JP/concepts/session-tool → `raw/docs/concepts/session-tool.md`
  - https://docs.openclaw.ai/ja-JP/concepts/channel-docking → `raw/docs/concepts/channel-docking.md`
  - https://docs.openclaw.ai/ja-JP/concepts/experimental-features → `raw/docs/concepts/experimental-features.md`
  - https://docs.openclaw.ai/ja-JP/concepts/qa-e2e-automation → `raw/docs/concepts/qa-e2e-automation.md`
  - https://docs.openclaw.ai/ja-JP/concepts/qa-matrix → `raw/docs/concepts/qa-matrix.md`
  - https://docs.openclaw.ai/ja-JP/start/bootstrapping → `raw/docs/start/bootstrapping.md`（**初の `start/` セクション**）
- 作成（sources）: [[sources/concepts/session]], [[sources/concepts/session-pruning]], [[sources/concepts/session-tool]], [[sources/concepts/channel-docking]], [[sources/concepts/experimental-features]], [[sources/concepts/qa-e2e-automation]], [[sources/concepts/qa-matrix]], [[sources/start/bootstrapping]]
- 作成（concepts）: [[concepts/session]], [[concepts/session-pruning]], [[concepts/session-tool]], [[concepts/channel-docking]], [[concepts/qa-automation]]（qa-e2e-automation と qa-matrix の 2 source を束ねる）
- 更新: [[concepts/context]]・[[sources/concepts/context]]（session/session-pruning の (未作成) を解消）, [[sources/concepts/agent-loop]]（system-prompt の (未作成) を解消）, [[index]]（Sources に `### start` 追加・略称 TTL/DM/QA/E2E/SUT 追加）, [[overview]]
- メモ:
  - 前回 dangling だった [[concepts/session]] と [[concepts/session-pruning]] を解消。
  - 粒度判断：`experimental-features`（フラグのカタログ）と `start/bootstrapping`（小さな初回手順）は **source のみ**で concept ページ化せず。QA は 2 source（qa-e2e-automation / qa-matrix）を 1 concept（[[concepts/qa-automation]]）に束ねた（qa-matrix・qa-e2e-automation は **source のみ**で別 concept は作らない）。
  - QA はメンテナー向け社内ツールのため、両 source 冒頭に「利用者には直接関係しない」旨を明記。図・ラスター画像の取り込みは無し。
  - 新規 dangling link（次回候補）: concepts/{mantis,multi-agent,compaction,memory,sandboxing,model-providers}, channels/{matrix,slack,channel-routing}, providers/{openai,anthropic}。

## [2026-06-14] ingest | concepts メモリ群 9ページ（memory hub＋engines＋search/active/dreaming/commitments/compaction）

- 取り込み: `Clippings/` の 9 clip を一括 ingest（すべて concepts）
  - .../concepts/memory → `raw/docs/concepts/memory.md`
  - .../concepts/memory-builtin → `raw/docs/concepts/memory-builtin.md`
  - .../concepts/memory-qmd → `raw/docs/concepts/memory-qmd.md`
  - .../concepts/memory-honcho → `raw/docs/concepts/memory-honcho.md`
  - .../concepts/memory-search → `raw/docs/concepts/memory-search.md`
  - .../concepts/active-memory → `raw/docs/concepts/active-memory.md`
  - .../concepts/dreaming → `raw/docs/concepts/dreaming.md`
  - .../concepts/commitments → `raw/docs/concepts/commitments.md`
  - .../concepts/compaction → `raw/docs/concepts/compaction.md`
- 作成（sources, 9）: [[sources/concepts/memory]], [[sources/concepts/memory-builtin]], [[sources/concepts/memory-qmd]], [[sources/concepts/memory-honcho]], [[sources/concepts/memory-search]], [[sources/concepts/active-memory]], [[sources/concepts/dreaming]], [[sources/concepts/commitments]], [[sources/concepts/compaction]]
- 作成（concepts, 6）: [[concepts/memory]]（ハブ。3 engine source を束ねる）, [[concepts/memory-search]], [[concepts/active-memory]], [[concepts/dreaming]], [[concepts/commitments]], [[concepts/compaction]]
- 更新: [[index]]（Sources/Concepts に新規・略称 BM25/FTS/MMR/GGUF）, [[overview]]（「メモリと長期記憶」段落）, および compaction/memory の (未作成) 注記を解消した 8 ページ（concepts: context-engine/session-pruning/session/agent-loop, sources: context/context-engine/agent-loop/agent-workspace）
- メモ:
  - **多数の dangling を一気に解消**：前回まで頻繁に参照されていた [[concepts/compaction]] と [[concepts/memory]] を作成。
  - 粒度判断：メモリの 3 バックエンド（builtin/qmd/honcho）は **source のみ**にし、[[concepts/memory]] ハブに「代表バックエンド」として束ねた（concept ページの乱立回避）。memory-search/active-memory/dreaming/commitments/compaction は独立 concept。
  - 図：active-memory と memory-search の Mermaid flowchart（インライン SVG）を ```mermaid で描き直し。dreaming/commitments 等は図なし。ラスター画像なし。
  - メモ内で言及した Plugin（Memory Wiki / LanceDB）は plugins/ カテゴリ未整備のため wikilink 化せず名称参照に留めた。
  - 新規 dangling（次回候補）: concepts/{multi-agent,sandboxing,streaming,queue,pairing,model-providers,mantis}, gateway/heartbeat, automation/cron-jobs, channels/*, providers/*, plugins/{memory-wiki,memory-lancedb}。

## [2026-06-14] ingest | concepts マルチエージェント群 4ページ（multi-agent / delegate-architecture / parallel-specialist-lanes / presence）

- 取り込み: `Clippings/` の 4 clip を一括 ingest（すべて concepts）
  - .../concepts/multi-agent → `raw/docs/concepts/multi-agent.md`
  - .../concepts/delegate-architecture → `raw/docs/concepts/delegate-architecture.md`
  - .../concepts/parallel-specialist-lanes → `raw/docs/concepts/parallel-specialist-lanes.md`
  - .../concepts/presence → `raw/docs/concepts/presence.md`
- 作成（sources, 4）: [[sources/concepts/multi-agent]], [[sources/concepts/delegate-architecture]], [[sources/concepts/parallel-specialist-lanes]], [[sources/concepts/presence]]
- 作成（concepts, 4）: [[concepts/multi-agent]]（ハブ）, [[concepts/delegate-architecture]], [[concepts/parallel-specialist-lanes]], [[concepts/presence]]
- 更新: [[index]]・[[overview]]、および [[concepts/multi-agent]] の (未作成) 注記を解消した 5 ページ（concepts: agent/session-tool, sources: agent/session-tool/session）
- メモ:
  - **頻繁に参照されていた [[concepts/multi-agent]] を作成**し、関連 dangling を一括解消。multi-agent をハブに、組織応用（delegate-architecture）・並列設計（parallel-specialist-lanes）を周辺概念として配置。
  - presence はやや独立した概念（Gateway 接続状況の軽量ビュー）として [[components/gateway]]/[[components/node]]/[[concepts/architecture]] に接続。
  - 図・ラスター画像の取り込みは無し（4 件とも図は本質的でない、または SVG 無し）。
  - 新規 dangling（次回候補）: concepts/{sandboxing,queue,heartbeat,channel-routing}, automation/{standing-orders,cron-jobs}, channels/*, providers/*。

## [2026-06-14] ingest | concepts メッセージ処理群 7ページ（messages / queue / queue-steering / streaming / progress-drafts / retry / message-lifecycle-refactor）

- 取り込み: `Clippings/` の 7 clip を一括 ingest（すべて concepts）
  - .../concepts/messages → `raw/docs/concepts/messages.md`
  - .../concepts/queue → `raw/docs/concepts/queue.md`
  - .../concepts/queue-steering → `raw/docs/concepts/queue-steering.md`
  - .../concepts/streaming → `raw/docs/concepts/streaming.md`
  - .../concepts/progress-drafts → `raw/docs/concepts/progress-drafts.md`
  - .../concepts/retry → `raw/docs/concepts/retry.md`
  - .../concepts/message-lifecycle-refactor → `raw/docs/concepts/message-lifecycle-refactor.md`
- 作成（sources, 7）: [[sources/concepts/messages]], [[sources/concepts/queue]], [[sources/concepts/queue-steering]], [[sources/concepts/streaming]], [[sources/concepts/progress-drafts]], [[sources/concepts/retry]], [[sources/concepts/message-lifecycle-refactor]]
- 作成（concepts, 5）: [[concepts/messages]]（ハブ）, [[concepts/queue]]（queue-steering を統合）, [[concepts/streaming]], [[concepts/progress-drafts]], [[concepts/retry]]
- 更新: [[index]]・[[overview]]、および queue/streaming の (未作成) を解消した 4 ページ（concepts: agent-loop/parallel-specialist-lanes, sources: agent-loop/parallel-specialist-lanes）
- メモ:
  - **頻繁に参照されていた [[concepts/queue]] と [[concepts/streaming]] を作成**し、関連 dangling を一括解消。
  - 粒度判断：`queue-steering`（queue モードのランタイム詳細）は **source のみ**にし [[concepts/queue]] に統合。`message-lifecycle-refactor`（送受信ライフサイクルの将来設計＝コントリビューター向け RFC）も **source のみ**にし、冒頭で「実装済み仕様ではなく目標設計」と明記。
  - 図：messages のパイプラインを Mermaid flowchart で作図（原典は ASCII）。streaming/progress-drafts 等のテキスト図は文章/箇条書きで再構成。ラスター画像なし。
  - これで `concepts` セクションはほぼ網羅。次は channels/providers/gateway 等の別セクション、または `/lint` での棚卸し。
  - 新規 dangling（次回候補）: concepts/{model-providers,model-failover,sandboxing,heartbeat,pairing,typing-indicators}, plugins/sdk-channel-turn, tools/{steer,thinking}, channels/*, providers/*。

## [2026-06-14] lint-fix | 自己矛盾リンクの解消（source-only ページへの concept リンク誤り）

- lint で検出した「concept ページを作らず source のみにしたページを `[[concepts/...]]` でリンクしていた」3 件を修正（リンク先を実在の source ページへ付け替え）：
  - `[[concepts/qa-matrix]]` → `[[sources/concepts/qa-matrix]]`（index・[[concepts/qa-automation]]・[[sources/concepts/qa-e2e-automation]]、計 4 箇所）
  - `[[concepts/qa-e2e-automation]]` → `[[sources/concepts/qa-e2e-automation]]`（[[sources/concepts/qa-matrix]]、2 箇所）
  - `[[concepts/queue-steering]]` → `[[sources/concepts/queue-steering]]`（[[sources/concepts/queue]]、1 箇所）
- 本ログの 8 件バッチのメモ（qa-matrix を「別 concept も用意」と誤記）を実態（source のみ）に修正。
- lint 結果サマリ: sources↔raw/docs 完全ミラー（38↔38）・孤立ページ 0・frontmatter type 欠落 0・component↔concept 相互リンク良好。残る dangling は未取り込みの将来ページのみ（参照多: sandboxing/pairing/model-providers/components-cli、未着手カテゴリ channels・providers）。

## [2026-06-14] ingest | gateway 設定・運用群 7ページ（gateway 運用手順書＋configuration 一式）— 初の gateway/ セクション

- 取り込み: `Clippings/` の 7 clip を一括 ingest（**初の `gateway/` セクション**）
  - .../gateway → `raw/docs/gateway/gateway.md`（セクションランディング。URL 単一セグメントのため slug=section＝`gateway/gateway`）
  - .../gateway/configuration → `raw/docs/gateway/configuration.md`
  - .../gateway/configuration-reference → `raw/docs/gateway/configuration-reference.md`
  - .../gateway/configuration-examples → `raw/docs/gateway/configuration-examples.md`
  - .../gateway/config-agents → `raw/docs/gateway/config-agents.md`
  - .../gateway/config-channels → `raw/docs/gateway/config-channels.md`
  - .../gateway/config-tools → `raw/docs/gateway/config-tools.md`
- 作成（sources, 7）: [[sources/gateway/gateway]], [[sources/gateway/configuration]], [[sources/gateway/configuration-reference]], [[sources/gateway/configuration-examples]], [[sources/gateway/config-agents]], [[sources/gateway/config-channels]], [[sources/gateway/config-tools]]
- 作成（concepts, 1）: [[concepts/configuration]]（設定の横断ハブ。6 source を束ねる）
- 更新: [[components/gateway]]（運用面＝install/restart/監視・ポート優先順位・OpenAI 互換・configuration を追補、sources に [[sources/gateway/gateway]] 追加）, [[index]]（Sources `### gateway` を 7 件で埋める・Concepts に configuration）, [[overview]]
- メモ:
  - **スキーマ拡張（命名）**：セクションランディングページ（URL が `/ja-JP/<section>` の単一セグメント）は `<section>/<section>.md` に置く方針を採用（今回 `gateway/gateway`）。sources↔raw は引き続き完全ミラー。
  - 粒度判断：configuration 系 6 source（configuration / -reference / -examples / config-agents / -channels / -tools）は **1 concept（[[concepts/configuration]]）に束ね**、各 source は source のみ。巨大なリファレンス（config-agents 1453 行・-reference 1308 行等）は**全フィールドを転記せず、トップレベル地図＋要点**に圧縮（一次情報源は `openclaw config schema`/`config.schema.lookup` と明記）。図・画像なし。
  - これで `concepts` ほぼ網羅＋`gateway/` 着手。次は channels/ providers/ や、頻出 dangling の pairing/sandbox/model-providers（いずれも gateway/ や concepts/ に原典あり）。

## [2026-06-14] ingest | gateway 認証・シークレット群 5ページ（authentication / trusted-proxy-auth / secrets / secrets-plan-contract ＋ root: auth-credential-semantics）

- 取り込み: `Clippings/` の 5 clip を一括 ingest
  - .../gateway/authentication → `raw/docs/gateway/authentication.md`
  - .../gateway/trusted-proxy-auth → `raw/docs/gateway/trusted-proxy-auth.md`
  - .../gateway/secrets → `raw/docs/gateway/secrets.md`
  - .../gateway/secrets-plan-contract → `raw/docs/gateway/secrets-plan-contract.md`
  - .../auth-credential-semantics → `raw/docs/auth-credential-semantics.md`（**初のルート直下＝section プレフィックス無しの top-level 文書**）
- 作成（sources, 5）: [[sources/gateway/authentication]], [[sources/gateway/trusted-proxy-auth]], [[sources/gateway/secrets]], [[sources/gateway/secrets-plan-contract]], [[sources/auth-credential-semantics]]
- 作成（concepts, 2）: [[concepts/authentication]]（認証 3 レイヤーの区別ハブ）, [[concepts/secrets]]（SecretRef）
- 更新: [[concepts/oauth]]（authentication への接続＝モデル認証の OAuth 部分と明記）, [[concepts/configuration]]（authentication/secrets を related/本文に）, [[components/gateway]]（gateway.auth モード・secrets リンク）, [[index]], [[overview]]
- メモ:
  - **重要な発見＝認証は 3 レイヤー**：① Gateway 接続認証（gateway.auth: token/password/trusted-proxy）② モデル認証（API キー/OAuth/auth-profiles）③ デバイスペアリング。原典 `gateway/authentication` は ② のリファレンスで、① は configuration＋trusted-proxy-auth にある。混同を防ぐため [[concepts/authentication]] を Mermaid 付きの区別ハブにした。
  - **スキーマ拡張（命名）**：`auth-credential-semantics` は URL に section が無い top-level 文書 → `raw/docs/auth-credential-semantics.md` と `sources/auth-credential-semantics.md`（doc_section: root）の**ルート直下**に置く方針を採用。index Sources に `### (root)` 見出しを新設。sources↔raw は引き続き完全ミラー。
  - 粒度判断：trusted-proxy-auth は source のみで [[concepts/authentication]] に、secrets-plan-contract と auth-credential-semantics も source のみでそれぞれ [[concepts/secrets]]/[[concepts/authentication]] に統合。図：trusted-proxy フローと authentication 3 レイヤーを Mermaid で作図。
  - 新規 dangling（次回候補）: concepts/pairing（接続認証/デバイス信頼で頻出）, providers/{openai,anthropic}, gateway/{security,remote,tailscale,sandboxing}。

## [2026-06-14] ingest | gateway 観測・診断・Heartbeat 群 9ページ

- 取り込み: `Clippings/` の 9 clip を一括 ingest
  - .../gateway/heartbeat → `raw/docs/gateway/heartbeat.md`
  - .../gateway/health → `raw/docs/gateway/health.md`
  - .../gateway/doctor → `raw/docs/gateway/doctor.md`
  - .../gateway/troubleshooting → `raw/docs/gateway/troubleshooting.md`
  - .../gateway/diagnostics → `raw/docs/gateway/diagnostics.md`
  - .../gateway/logging → `raw/docs/gateway/logging.md`
  - .../gateway/prometheus → `raw/docs/gateway/prometheus.md`
  - .../gateway/opentelemetry → `raw/docs/gateway/opentelemetry.md`
  - .../logging → `raw/docs/logging.md`（**root の top-level 文書**、auth-credential-semantics に続く 2 例目）
- 作成（sources, 9）: [[sources/gateway/heartbeat]], [[sources/gateway/health]], [[sources/gateway/doctor]], [[sources/gateway/troubleshooting]], [[sources/gateway/diagnostics]], [[sources/gateway/logging]], [[sources/gateway/prometheus]], [[sources/gateway/opentelemetry]], [[sources/logging]]
- 作成（concepts, 4）: [[concepts/heartbeat]], [[concepts/logging]], [[concepts/observability]]（Prometheus/OTel）, [[concepts/diagnostics]]（health/doctor/troubleshooting/export）
- 更新: [[concepts/presence]]・[[sources/concepts/presence]]（heartbeat の (未作成) 解消）, [[concepts/commitments]]・[[concepts/dreaming]]（related に heartbeat）, [[components/gateway]]（観測・運用リンク追加）, [[index]], [[overview]]
- メモ:
  - **頻出 dangling の [[concepts/heartbeat]] を作成**して解消（commitments/dreaming/presence/config 等から参照）。
  - 粒度判断：観測/診断 9 docs を 4 concept に整理 — `logging`（ログ信号）/ `observability`（Prometheus+OTel のテレメトリ・エクスポート）/ `diagnostics`（health+doctor+troubleshooting+export）/ `heartbeat`。ログは 2 source（root `/logging` の利用者向け＋`gateway/logging` の内部）を `logging` に束ねた。巨大な troubleshooting(667)/doctor(511)/opentelemetry(346) は全転記せず症状/信号の地図に圧縮。
  - **root 文書 2 例目**：`/ja-JP/logging` を `raw/docs/logging.md`＋`sources/logging.md`（doc_section: root）へ。index `### (root)` に追加。sources↔raw 完全ミラー維持。
  - 図・ラスター画像なし（メトリクス表等はテキスト/コード列挙で代替）。
  - 新規 dangling（次回候補）: concepts/{pairing,timezone}, diagnostics/flags, automation/{cron-jobs,tasks}, plugins/codex-harness, providers/*, gateway/{security,remote,tailscale,sandboxing}。

## [2026-06-14] ingest | gateway 運用詳細 3ページ（gateway-lock / multiple-gateways / background-process）

- 取り込み: `Clippings/` の 3 clip を一括 ingest（すべて gateway/）
  - .../gateway/gateway-lock → `raw/docs/gateway/gateway-lock.md`
  - .../gateway/multiple-gateways → `raw/docs/gateway/multiple-gateways.md`
  - .../gateway/background-process → `raw/docs/gateway/background-process.md`
- 作成（sources, 3）: [[sources/gateway/gateway-lock]], [[sources/gateway/multiple-gateways]], [[sources/gateway/background-process]]
- concept: 新規作成なし（いずれも狭い運用/ツール詳細のため source のみ）
- 更新: [[components/gateway]]（不変条件にロック機構・複数 Gateway・exec/process を追補）, [[index]]
- メモ:
  - 粒度判断：gateway-lock（1 ホスト 1 Gateway の強制）・multiple-gateways（レスキューボット等の分離運用）・background-process（`exec`/`process` ツール）は、いずれも既存概念（[[concepts/architecture]] の不変条件 / [[concepts/configuration]] の分離 / [[concepts/agent]] のツール）に紐づく**運用詳細**なので concept ページは作らず source のみ。background-process は将来 `tools/` セクション取り込み時に tools 概念へ接続可能。
  - 図・画像なし。overview は変更なし（新カテゴリでないため）。
  - これで `gateway/` セクションは設定・運用・認証・シークレット・観測/診断・ロック/プロセスまでほぼ網羅。残りは gateway/{security,remote,tailscale,sandboxing,bonjour,discovery,heartbeat 済,...}。

## [2026-06-14] ingest | gateway セキュリティ・サンドボックス群 7ページ（初の gateway/security/ ネスト）

- 取り込み: `Clippings/` の 7 clip を一括 ingest
  - .../gateway/security → `raw/docs/gateway/security.md`
  - .../gateway/sandboxing → `raw/docs/gateway/sandboxing.md`
  - .../gateway/sandbox-vs-tool-policy-vs-elevated → `raw/docs/gateway/sandbox-vs-tool-policy-vs-elevated.md`
  - .../gateway/openshell → `raw/docs/gateway/openshell.md`
  - .../gateway/operator-scopes → `raw/docs/gateway/operator-scopes.md`
  - .../gateway/security/audit-checks → `raw/docs/gateway/security/audit-checks.md`（**初の 3 階層ネスト `gateway/security/`**）
  - .../gateway/security/secure-file-operations → `raw/docs/gateway/security/secure-file-operations.md`
- 作成（sources, 7）: [[sources/gateway/security]], [[sources/gateway/sandboxing]], [[sources/gateway/sandbox-vs-tool-policy-vs-elevated]], [[sources/gateway/openshell]], [[sources/gateway/operator-scopes]], [[sources/gateway/security/audit-checks]], [[sources/gateway/security/secure-file-operations]]
- 作成（concepts, 2）: [[concepts/sandboxing]]（3 制御の区別ハブを含む）, [[concepts/security]]（パーソナルアシスタント信頼モデルのハブ）
- 更新: sandboxing の (未作成) を解消した 7 ページ（concepts: multi-agent/configuration/delegate-architecture, sources: agent-workspace/multi-agent/delegate-architecture/configuration-examples）, [[components/gateway]]（セキュリティ関連リンク追加）, [[index]], [[overview]]
- メモ:
  - **頻出 dangling の [[concepts/sandboxing]] を作成して解消**（agent-workspace/multi-agent/delegate-architecture/configuration 等から多数参照）。
  - **重要な発見＝3 つの制御**：サンドボックス（どこで実行）/ツールポリシー（どのツール）/昇格（exec 外し）は別物。認証 3 レイヤーと同じく、混同事故を防ぐため [[concepts/sandboxing]] に区別表を置いた。openshell は backend の 1 つとして source のみで sandboxing に統合。
  - 粒度判断：security 系 7 docs を 2 concept（sandboxing / security）に整理。operator-scopes・audit-checks・secure-file-operations は source のみで [[concepts/security]] に統合。巨大な security(1210)/audit-checks(checkId 多数)/sandboxing(501) は全転記せず脅威モデル・カテゴリ地図に圧縮。
  - **スキーマ拡張：初の 3 階層ネスト** `gateway/security/<slug>` を `raw/docs/gateway/security/` と `sources/gateway/security/`（doc_section: gateway）にミラー。サブフォルダミラーが任意深さで機能することを確認。sources↔raw 完全ミラー維持。
  - 新規 dangling（次回候補）: concepts/pairing（最頻出・接続認証/Node で重要）, channels/channel-routing, tools/{elevated,exec-approvals,multi-agent-sandbox-tools}, providers/*, gateway/{remote,tailscale,bonjour,discovery}。

## [2026-06-14] ingest | gateway 接続・API・ローカルモデル群 12ページ（network/protocol/pairing/discovery/http-api/local-models）

- 取り込み（12 docs。1 件は root の `/ja-JP/network`、他 11 件は `/ja-JP/gateway/...`）:
  - https://docs.openclaw.ai/ja-JP/network
  - https://docs.openclaw.ai/ja-JP/gateway/protocol
  - https://docs.openclaw.ai/ja-JP/gateway/pairing
  - https://docs.openclaw.ai/ja-JP/gateway/discovery
  - https://docs.openclaw.ai/ja-JP/gateway/bonjour
  - https://docs.openclaw.ai/ja-JP/gateway/bridge-protocol
  - https://docs.openclaw.ai/ja-JP/gateway/openai-http-api
  - https://docs.openclaw.ai/ja-JP/gateway/openresponses-http-api
  - https://docs.openclaw.ai/ja-JP/gateway/tools-invoke-http-api
  - https://docs.openclaw.ai/ja-JP/gateway/local-models
  - https://docs.openclaw.ai/ja-JP/gateway/local-model-services
  - https://docs.openclaw.ai/ja-JP/gateway/cli-backends
- 作成（sources, 12）: [[sources/network]]（root）, [[sources/gateway/protocol]], [[sources/gateway/pairing]], [[sources/gateway/discovery]], [[sources/gateway/bonjour]], [[sources/gateway/bridge-protocol]], [[sources/gateway/openai-http-api]], [[sources/gateway/openresponses-http-api]], [[sources/gateway/tools-invoke-http-api]], [[sources/gateway/local-models]], [[sources/gateway/local-model-services]], [[sources/gateway/cli-backends]]
- 作成（concepts, 4）: [[concepts/pairing]]（最頻出 dangling を解消）, [[concepts/discovery]]（Bonjour/tailnet/SSH の Mermaid フローを含む）, [[concepts/http-api]]（3 エンドポイント比較表）, [[concepts/local-models]]（3 つの取り込み方）
- 更新: [[concepts/pairing]] の (未作成) を解消した 5 ページ（concepts: authentication/architecture, sources: concepts/architecture・gateway/operator-scopes・gateway/trusted-proxy-auth）, [[components/gateway]]（http-api/discovery/local-models リンク追加）, [[components/node]]（discovery/pairing 強化）, [[concepts/architecture]]（protocol/discovery/http-api 追加）, [[index]], [[overview]]
- メモ:
  - **最頻出 dangling の [[concepts/pairing]] を作成**（operator-scopes/architecture/authentication/trusted-proxy-auth 等から多数参照）。認証 3 レイヤーの③として位置づけ、「トークン発行であってコマンドサーフェス固定ではない」「2026.3.31+ はノードペアリング承認まで node コマンド無効」を明示。
  - 粒度判断：12 docs を 4 concept に整理。`protocol` は [[concepts/architecture]] の詳細契約として source のみ、`bonjour`/`bridge-protocol`/`network` は [[concepts/discovery]] に統合、3 つの HTTP API は [[concepts/http-api]] に集約、`local-model-services`/`cli-backends` は [[concepts/local-models]] に統合（cli-backends は [[concepts/agent-runtimes]]/[[concepts/retry]] にもリンク）。
  - 大きめの巨大 doc（protocol 612 行・openai-http-api・local-models）は全転記せず、契約面・セキュリティ境界・運用要点に圧縮。HTTP API の「完全オペレーターアクセス＝狭い x-openclaw-scopes を無視」と `/tools/invoke` のハード拒否リスト（exec/spawn/shell/fs_*/gateway/nodes/cron）を強調。
  - `model-providers` 概念は未作成のため、ローカルモデルの provider/model 層は既存の [[concepts/agent-runtimes]] に接続（将来 model-providers ハブを作る候補）。
  - index 略称リダイレクト追加（DNS-SD/mDNS/RCE）、GGUF を memory-search→local-models に修正。sources↔raw 81↔81 完全ミラーを確認。
  - 新規 dangling（次回候補）: providers/{openai,lmstudio,ollama,inferrs}, channels/pairing, gateway/{remote,tailscale}, tools/acp-agents, concepts/model-providers。

## [2026-06-14] ingest | nodes/tools/security 群 18ページ（Node 拡充・メディア理解・音声・リモートアクセス・脅威モデル）

- 取り込み（18 docs。新規 3 トップレベルセクション nodes/ tools/ security/ ＋ gateway/ 3 件）:
  - https://docs.openclaw.ai/ja-JP/nodes ・ /nodes/camera ・ /nodes/audio ・ /nodes/images ・ /nodes/media-understanding ・ /nodes/talk ・ /nodes/voicewake ・ /nodes/location-command ・ /nodes/troubleshooting
  - https://docs.openclaw.ai/ja-JP/gateway/remote ・ /gateway/remote-gateway-readme ・ /gateway/tailscale
  - https://docs.openclaw.ai/ja-JP/tools/tts ・ /tools/video-generation
  - https://docs.openclaw.ai/ja-JP/security/network-proxy ・ /security/THREAT-MODEL-ATLAS ・ /security/formal-verification ・ /security/CONTRIBUTING-THREAT-MODEL
- 作成（sources, 18）: nodes/{nodes,camera,audio,images,media-understanding,talk,voicewake,location-command,troubleshooting}, gateway/{remote,remote-gateway-readme,tailscale}, tools/{tts,video-generation}, security/{network-proxy,threat-model-atlas,formal-verification,contributing-threat-model}
- 作成（concepts, 4）: [[concepts/media-understanding]]（受信メディア→テキスト・入力側）, [[concepts/voice]]（Talk/TTS/Voice Wake・Mermaid フロー）, [[concepts/remote-access]]（SSH/Tailscale/bind モード）, [[concepts/threat-model]]（MITRE ATLAS 5 信頼境界・形式検証・Mermaid）
- 更新: [[components/node]] を大幅拡充（コマンドサーフェス・ノードホスト・3 ゲート・Mac Node）, [[components/gateway]]（remote-access/voice/media-understanding/threat-model リンク）, [[concepts/discovery]]（remote-access）, [[concepts/messages]]（media-understanding/voice）, [[concepts/security]]（threat-model/network-proxy）, [[index]], [[overview]]。discovery.md の `sources/gateway/remote`（未作成）を解消。
- メモ:
  - **クリップのタイトルが Web Clipper の誤名（「カメラキャプチャ 1/2/3」「テキスト読み上げ 1」等）**だったため、すべて frontmatter の `source:` URL を真実として section/slug を導出。
  - 粒度判断：18 docs を 4 concept＋node 大幅拡充に整理。nodes/ の能力ページ（camera/audio/images/location/troubleshooting）は [[components/node]] 配下の source。`media-understanding`＝入力（受信メディア要約）、`voice`＝Talk/TTS/voicewake をまとめる出力＋リアルタイム、`remote-access`＝gateway/remote+tailscale+remote-gateway-readme、`threat-model`＝security/ 4 docs（network-proxy は [[concepts/security]] 配下の source）。
  - `tools/video-generation` は単独 doc のため source のみ（[[concepts/session-tool]]/tasks に接続）。`security/threat-model-atlas`（619 行）は全 T-XXX 表を転記せず 5 信頼境界＋最上位リスク＋攻撃チェーンに圧縮、`tts`（986 行）は 14 プロバイダー＋ペルソナ＋出力の要点に圧縮。
  - スキーマ：新トップレベルセクション `nodes/` `tools/` `security/` を raw と sources にミラー（既存 `gateway/security/` ネストとは別物の `security/` ルートセクション）。`THREAT-MODEL-ATLAS`/`CONTRIBUTING-THREAT-MODEL` の大文字 URL スラグは kebab-case（threat-model-atlas / contributing-threat-model）に正規化して raw↔sources を一致させた。sources↔raw 完全ミラーを確認。
  - 新規 dangling（次回候補）: components/clawhub（脅威モデルのサプライチェーン）, channels/{whatsapp,location}, providers/{google,elevenlabs,deepgram,...}, tools/exec-approvals, platforms/{ios,android,mac}。

## [2026-06-14] ingest | web 群 4ページ（Control UI / WebChat / TUI＝クライアント UI 3 構成要素）

- 取り込み（web/ セクション。4 ユニーク docs。ダッシュボード.md は webchat の重複再 clip だったため除外）:
  - https://docs.openclaw.ai/ja-JP/web ・ /web/control-ui ・ /web/webchat ・ /web/tui
- 作成（sources, 4）: web/{web,control-ui,webchat,tui}
- 作成（components, 3）: [[components/control-ui]]（ブラウザー管理ダッシュボード）, [[components/webchat]]（ネイティブチャット UI）, [[components/cli]]（CLI＋対話 TUI、既存の 8 dangling を解消）
- 更新: [[components/gateway]]（クライアント面 control-ui/webchat/cli リンク・cli の未作成解消）, [[components/node]]（cli の未作成解消）, [[concepts/architecture]]（クライアント UI を実ページに）, [[concepts/voice]]（ブラウザー Talk→control-ui）, [[index]], [[overview]]
- メモ:
  - **`ダッシュボード.md` は `ウェブチャット.md` と本文が同一**（title/description のみ差分、source URL は両方 web/webchat）の Web Clipper 重複。webchat を 1 回だけ ingest し、重複クリップは raw に残さず削除（raw↔sources 1:1 を維持）。
  - 粒度判断：web/ の 3 UI（control-ui/webchat/tui）はいずれも landmark な構成要素として独立ページ化。新概念は作らず、3 つとも上位概念 [[concepts/architecture]]（WS クライアント）に接続。web ランディングは [[components/control-ui]] 配下の source。
  - **`components/cli` を新設**：TUI doc を主軸に、wiki 全体で散在する `openclaw <cmd>` の入口を兼ねる。これで既存 8 ファイルの `[[components/cli]]（未作成）` 系 dangling を解消（cli/ セクション docs は今後拡充）。
  - Control UI の巨大 doc（481 行）は全機能を転記せず「何ができるか」のカテゴリ＋認証/ペアリング/非セキュア HTTP の要点に圧縮。
  - sources↔raw 完全ミラーを確認。新規 dangling（次回候補）: components/clawhub, web/{dashboard,a2ui}, channels/*, providers/*, cli/* セクション。

## [2026-06-14] ingest | tools/plugins 群 15ページ（Plugin System・ClawHub・Codex ハーネス・音声/メモリ Plugin・最初のチャネル）

- 取り込み（tools/ 2 件＋plugins/ 13 件）:
  - https://docs.openclaw.ai/ja-JP/tools ・ /tools/plugin
  - https://docs.openclaw.ai/ja-JP/plugins/manage-plugins ・ /bundles ・ /community ・ /codex-harness ・ /codex-computer-use ・ /codex-native-plugins ・ /google-meet ・ /voice-call ・ /memory-lancedb ・ /memory-wiki ・ /webhooks ・ /oc-path ・ /zalouser
- 作成（sources, 15）: tools/{tools,plugin}, plugins/{manage-plugins,bundles,community,codex-harness,codex-computer-use,codex-native-plugins,google-meet,voice-call,memory-lancedb,memory-wiki,webhooks,oc-path,zalouser}
- 作成（components, 2）: [[components/plugin-system]]（拡張機構の landmark、ネイティブ/バンドル・スロット・登録 API）, [[components/clawhub]]（Plugin/Skill マーケットプレイス。専用 clawhub/ docs は未取り込みのため言及範囲から構成）
- 作成（channels, 1）: [[channels/zalouser]]（最初のチャネルページ。Zalo 個人アカウント・非公式）
- 更新: [[concepts/threat-model]]（clawhub の未作成解消＋⑤境界に component リンク）, [[concepts/memory]]（LanceDB/Memory Wiki バックエンドを実ソースに）, [[concepts/agent-runtimes]]（Codex ハーネス 3 ソース）, [[concepts/voice]]（voice-call/google-meet 拡張）, [[components/gateway]]（plugin-system/clawhub リンク）, [[index]], [[overview]]
- メモ:
  - **`ダッシュボード.md` の前例と同様、重複や別名に注意**。今回は重複なし。15 docs すべて固有。
  - 粒度判断：拡張機構そのものは **components/plugin-system** に集約（tools/plugin＋manage-plugins＋bundles＋community が支えるソース）。個別 Plugin は `sources/plugins/<slug>` に置き、ドメイン概念へ接続——Codex 3 件→[[concepts/agent-runtimes]]、memory 2 件→[[concepts/memory]]、voice-call/google-meet→[[concepts/voice]]、webhooks→[[concepts/queue]]/secrets、oc-path→[[components/cli]]。新概念は作らず。
  - **ClawHub を component 化**：plugin/community/manage と既存 [[concepts/threat-model]] から繰り返し参照される landmark で、recurring dangling を解消。専用 `clawhub/` セクション docs を取り込んだら拡充する（その旨ページに明記）。
  - **最初の channels/ ページ**：zalouser は実体が channel（`channels.zalouser` 設定）なので channels/ に起こした。official `zalo` は将来別ページ。channel-routing 概念は未作成のため messages/gateway/pairing に接続。
  - 巨大 doc（google-meet 1441・voice-call 853・tools/plugin 493）は全転記せず、要点（モード/プロバイダー/認証/セキュリティ）に圧縮。sources↔raw 完全ミラーを確認。
  - 新規 dangling（次回候補）: concepts/{model-providers,channel-routing}, providers/*, channels/{slack,telegram,whatsapp,discord,...}, plugins/{building-plugins,sdk-overview,manifest}, clawhub/ セクション, automation セクション。

## [2026-06-14] ingest | Skills・スラッシュコマンド・Plugin SDK 群 11ページ（concepts: skills / slash-commands）

- 取り込み（tools/ 4・plugins/ 6・root 1）:
  - https://docs.openclaw.ai/ja-JP/tools/skills ・ /tools/creating-skills ・ /tools/skills-config ・ /tools/slash-commands
  - https://docs.openclaw.ai/ja-JP/plugins/building-plugins ・ /hooks ・ /sdk-channel-plugins ・ /sdk-provider-plugins ・ /cli-backend-plugins ・ /adding-capabilities
  - https://docs.openclaw.ai/ja-JP/prose
- 作成（sources, 11）: tools/{skills,creating-skills,skills-config,slash-commands}, plugins/{building-plugins,hooks,sdk-channel-plugins,sdk-provider-plugins,cli-backend-plugins,adding-capabilities}, prose（root）
- 作成（concepts, 2）: [[concepts/skills]]（`SKILL.md` 指示パック＝ツール/Skills/Plugins 三分法の中段）, [[concepts/slash-commands]]（コマンド/ディレクティブ/インラインショートカット・オーナー専用・ネイティブ/テキスト）
- 更新: [[components/plugin-system]]（Plugin SDK セクション＋hooks/building/SDK ソース＋prose）, [[concepts/agent-runtimes]]（provider/cli-backend SDK・adding-capabilities）, [[concepts/multi-agent]]（OpenProse）, [[concepts/messages]]（slash-commands）, [[components/cli]]（slash-commands）, [[sources/tools/tools]]（skills/slash-commands リンク）, [[index]], [[overview]]
- メモ:
  - 粒度判断：Skills 3 docs を **concept skills** に集約（三分法の中段。tools/tools が予告していた「Skills」ピラーを実体化）。slash-commands は全サーフェス横断の制御面なので **concept slash-commands** を新設。Plugin SDK 6 docs（building/hooks/channel/provider/cli-backend/adding-capabilities）は **[[components/plugin-system]] 配下の source**（新概念は作らず、plugin-system に「Plugin SDK」セクションを追加）。
  - OpenProse は root レベル `/prose`（`sources/prose.md`・doc_section: root）。OpenClaw 専用でなく Plugin として載るワークフロー DSL なので multi-agent/parallel-specialist-lanes/plugin-system に接続。
  - SDK channel/provider ガイドは将来の `concepts/channel-routing`・`concepts/model-providers` ハブの布石（本バッチでは作らず、実チャネル/プロバイダー ingest 時に起こす）。
  - 巨大 doc（provider 671・channel 556・slash-commands 449・building 407）は要点圧縮。sources↔raw 完全ミラーを確認。
  - 新規 dangling（次回候補）: concepts/{channel-routing,model-providers,markdown-formatting}, channels/*, providers/*, automation/{hooks,...}, tools/{btw,steer,lobster,trajectory,acp-agents}, plugins/{manifest,sdk-overview,architecture,skill-workshop}。

## [2026-06-14] ingest | tools/automation 群 26ページ（concepts: exec / automation / cron / tasks / hooks）

- 取り込み（tools/ 21・automation/ 6。automation/ は新トップレベルセクション。ブラウザーはバッチ途中に追加された 27 件目）:
  - tools: browser, exec, exec-approvals, exec-approvals-advanced, elevated, code-execution, apply-patch, diffs, tool-search, tokenjuice, loop-detection, lobster, llm-task, btw, trajectory, thinking, media-overview, image-generation, music-generation, pdf, reactions
  - automation: automation, cron-jobs, tasks, taskflow, hooks, standing-orders
- 作成（sources, 26）: 上記すべて（tools/* と automation/*）
- 作成（concepts, 5）: [[concepts/exec]]（exec/process/code_execution＋承認＋昇格の4レイヤー・Mermaid）, [[concepts/automation]]（8 仕組みの判断ハブ）, [[concepts/cron]]（正確なスケジュール）, [[concepts/tasks]]（タスク台帳＋Task Flow）, [[concepts/hooks]]（内部 HOOK.md フック。Plugin SDK フックと区別）
- 更新: [[concepts/session-tool]]（ツールカタログ/exec/メディアへの導線）, [[concepts/sandboxing]]（exec リンク＝3 制御の実行時具体化）, [[concepts/heartbeat]]/[[concepts/commitments]]（automation 内の位置づけ）, [[components/gateway]]（automation/exec/plugin-system）, [[index]], [[overview]]
- メモ:
  - 粒度判断：**automation/ を新セクション化**。8 仕組みのハブ概念 automation＋サブ概念 cron/tasks/hooks を新設（heartbeat/commitments は既存）。standing-orders/taskflow は source（standing-orders→automation、taskflow→tasks）。memory ハブ＋バックエンドや messages ハブ＋サブ概念と同じパターン。
  - **exec を概念化**：exec/exec-approvals/exec-approvals-advanced/elevated/code-execution の 5 docs を 1 concept に集約（ツールポリシー→昇格→承認→ホストの 4 レイヤーを Mermaid 化）。sandboxing の「3 制御」を実行時に具体化する面として接続。elevated は sandboxing の昇格制御そのもの。
  - **内部フック vs Plugin SDK フック**：automation/hooks（HOOK.md スクリプト）と plugins/hooks（コード介入）は別系統。concept hooks の冒頭で明示区別。
  - 残り 20 tools は大半が [[concepts/session-tool]] 配下の source。メディア生成（image/music/video）は [[concepts/tasks]] のバックグラウンドタスクに、media-overview がハブ。code_execution は xai Plugin 製のリモート Python（ローカル exec と認可境界が別）。
  - 巨大 doc（cron 483・diffs 450・exec-approvals 403・lobster 395・tasks 370）は要点圧縮。automation landing の巨大 SVG 判断フローは表に再構成。sources↔raw 完全ミラーを確認。
  - 新規 dangling（次回候補）: providers/xai ほか providers/*, channels/*, concepts/{channel-routing,model-providers,markdown-formatting}, tools/{steer,web-fetch,browser,subagents,acp-agents}, clawhub/, cli/。

## [2026-06-14] ingest | Web 検索/取得・マルチエージェント/ACP 群 20ページ（concepts: web-search / acp）

- 取り込み（すべて tools/）:
  - web: web, web-fetch, brave-search, duckduckgo-search, exa-search, firecrawl, gemini-search, grok-search, kimi-search, minimax-search, ollama-search, perplexity-search, searxng-search, tavily
  - multi-agent/acp: subagents, agent-send, multi-agent-sandbox-tools, steer, acp-agents, acp-agents-setup
- 作成（sources, 20）: 上記すべて（tools/*）
- 作成（concepts, 2）: [[concepts/web-search]]（web_search/x_search/web_fetch ＋ 12 プロバイダーのハブ）, [[concepts/acp]]（Agent Client Protocol＝外部コーディングハーネスを丸ごと実行するランタイム）
- 更新: [[concepts/agent-runtimes]]（ACP を「もう一つのランタイム」として）, [[concepts/session-tool]]（web-search/subagents/acp 導線）, [[concepts/multi-agent]]（subagents/multi-agent-sandbox-tools/acp）, [[concepts/queue]]（steer）, [[concepts/security]]（web-search の SSRF 面・exec）, [[components/cli]]（agent-send）, [[index]], [[overview]]。cli-backends の `acp-agents`（未作成）を解消。SSRF 略称リダイレクトの重複を統合。
- メモ:
  - 粒度判断：**web-search を概念化**。web 概要＋web-fetch ＋ 12 検索プロバイダー（brave/ddg/exa/firecrawl/gemini/grok/kimi/minimax/ollama/perplexity/searxng/tavily）を 1 concept に集約。プロバイダーは構造化スニペット系と AI 合成＋引用系の 2 系統。各プロバイダーは plugin として実装され、provider 比較表・自動検出順を web overview から再構成（個別プロバイダー source は概要の記述から簡潔に）。
  - **ACP を概念化**：agent-runtimes の中で PI/Codex/CLI バックエンドと並ぶ「外部ハーネス完全ランタイム」。CLI バックエンド（テキスト軽量）との対比を明示。acp-agents/acp-agents-setup の 2 source。
  - subagents/steer/multi-agent-sandbox-tools は既存 multi-agent/queue/sandboxing 配下の source。agent-send は `openclaw agent` CLI なので [[components/cli]] 配下（A2A メッセージングではない点に注意）。
  - 巨大 doc（acp-agents 669・subagents 492・web 430・multi-agent-sandbox-tools 408）は要点圧縮。sources↔raw 完全ミラーを確認。
  - 新規 dangling（次回候補）: concepts/{channel-routing,model-providers}, providers/*, channels/*, tools/{x_search 専用は xai に内包}, cli/, clawhub/。

## [2026-06-14] ingest | channels/providers/platforms 群 23ページ（concepts: channel-routing / model-providers / model-failover）

- 取り込み（新セクション channels/ providers/ platforms/ announcements/ ＋ concepts/）:
  - concepts: model-providers, model-failover, models
  - channels: channels(landing), discord, slack, telegram, whatsapp
  - providers: providers(landing), models(quickstart), anthropic, openai, google, ollama, bedrock, litellm
  - platforms: platforms(landing), macos, ios, android, windows, linux
  - announcements: bluebubbles-imessage
- 作成（sources, 23）: 上記すべて
- 作成（concepts, 3）: [[concepts/channel-routing]]（25+ チャットチャネルのハブ）, [[concepts/model-providers]]（`provider/model`・50+ プロバイダーのハブ）, [[concepts/model-failover]]（認証ローテーション＋フォールバック・Mermaid）
- 作成（channels, 4）: [[channels/discord]], [[channels/slack]], [[channels/telegram]], [[channels/whatsapp]]
- 作成（providers, 6）: [[providers/anthropic]], [[providers/openai]], [[providers/google]], [[providers/ollama]], [[providers/bedrock]], [[providers/litellm]]
- 更新: [[components/gateway]]（channel-routing/model-providers/各プロバイダー）, [[concepts/agent-runtimes]]（model-providers/model-failover、provider ページ）, 多数の (未作成) 解消（model-providers/channel-routing/providers/openai 等を sed で一括）。`channels/channel-routing` の誤ディレクトリ参照を `concepts/channel-routing` に修正。config-channels の「個別チャネル未作成」注記を更新。[[index]], [[overview]]
- メモ:
  - **過去最大のバッチ。待望の channels/・providers/ セクションと新 platforms/ セクションが立ち上がった。** 2 ハブ概念（channel-routing・model-providers）はこれまで多数の (未作成) で参照されてきたもので、ここで解消。
  - 粒度判断：チャネル/プロバイダーは「source（doc 解説、raw ミラー）＋ channels|providers ページ（curated 統合ページ）」の二層（concepts/architecture と sources/concepts/architecture の関係に同じ）。統合ページは簡潔なハブにし、詳細は source に。
  - 巨大 doc（Discord 1580・Slack 1377・Telegram 1024・Ollama 1199・OpenAI 963・model-providers 732・WhatsApp 665）は全設定を転記せず、トランスポート・認証・グループ挙動・distinctive feature に圧縮。
  - platforms/ は新セクションだが概念を作らず、source が既存 component（node/gateway/control-ui/cli）に接続（macOS=Mac Node＋UI ホスト、iOS/Android=モバイルノード、Windows/Linux=Gateway 運用）。
  - announcements/ も新セクション（変更告知）。bluebubbles→imsg の iMessage 移行を記録。
  - sources↔raw 完全ミラーを確認。providers/lmstudio は本バッチ外で (未作成) のまま（LM Studio は今回未取り込み）。
  - 新規 dangling（次回候補）: providers/{lmstudio,openrouter,groq,mistral,xai,deepseek,...}, channels/{signal,imessage,matrix,feishu,...}, channels/groups, install/ セクション, cli/ セクション, clawhub/。

## [2026-06-15] ingest | channels 横断群 9ページ（concept: groups）

- 取り込み（すべて channels/）:
  - https://docs.openclaw.ai/ja-JP/channels/channel-routing ・ /groups ・ /broadcast-groups ・ /access-groups ・ /group-messages ・ /pairing ・ /location ・ /troubleshooting ・ /qa-channel
- 作成（sources, 9）: channels/{channel-routing, groups, broadcast-groups, access-groups, group-messages, pairing, location, troubleshooting, qa-channel}
- 作成（concepts, 1）: [[concepts/groups]]（グループチャット挙動のハブ＝メンションゲート/許可リスト/アクセスグループ/ブロードキャストの 4 軸表）
- 更新: [[concepts/channel-routing]]（channel-routing/groups/troubleshooting/location ソース＋groups 概念）, [[concepts/pairing]]（channels/pairing ＝ DM＋Node の統合入口を冒頭に反映）, [[concepts/qa-automation]]（qa-channel ソース）, [[index]], [[overview]]。`channels/location`・`channels/pairing` の誤参照（sources でなく channels 扱い）を修正。
- メモ:
  - 粒度判断：前バッチで作った [[concepts/channel-routing]] の下位に **concept groups を新設**（messages ハブの下の queue/streaming と同じパターン）。groups/broadcast-groups/access-groups/group-messages の 4 source を集約。channel-routing 専用 doc・location・troubleshooting は channel-routing 概念の source、pairing は [[concepts/pairing]]、qa-channel は [[concepts/qa-automation]] に接続。
  - **channels/pairing は DM＋Node の統合ペアリング doc**で、これまで gateway/pairing（Node 寄り）しか無かった pairing 概念に「DM ペアリング」の視点を補った。
  - broadcast-groups（実験的）は 1 グループに複数エージェント（parallel/sequential・セッション分離）で、[[concepts/multi-agent]]/[[concepts/parallel-specialist-lanes]] と接続。
  - 巨大 doc（groups 493・broadcast-groups 483）は要点圧縮。sources↔raw 完全ミラーを確認。
  - 新規 dangling（次回候補）: 残る個別 channels/{signal,imessage,matrix,feishu,...}・providers/{lmstudio,openrouter,groq,...}, install/, cli/, clawhub/。

## [2026-06-15] lint-fix | リンクバグ 2 件修正

- lint 実施（208 sources / 60 concepts / 7 components / 5 channels / 6 providers）。sources↔raw 208↔208 完全ミラー・孤立ページ ゼロを確認。
- 修正:
  - **experimental-features の誤パス（4 箇所）**：`[[concepts/experimental-features]]` → `[[sources/concepts/experimental-features]]`（concept ページは存在せず source のみ）。対象 `sources/gateway/config-tools.md`（3）・`sources/gateway/troubleshooting.md`（1）。
  - **[[channels/zalouser]] の channel-routing 未リンク**：channel-routing 概念より前に作成された最初のチャネルページ。frontmatter `related` と本文に [[concepts/channel-routing]] を追加（スキーマ規約「channels→channel-routing」に準拠）。
- 未対応（正当な未作成 TODO マーカー・ingest 待ち）: concepts/mantis（QA の修正前後ライブ検証ツール・4 参照）, channels/{matrix,imessage}, providers/{xai,lmstudio,elevenlabs}。これらはスキーマが許容する dangling で、対応 doc を ingest した時点で解消する。

## [2026-06-15] schema-update | 非公式記事（ブログ）の ingest 対応

- 背景：ユーザーが `raw/articles/` に非公式ブログ記事を配置して取り込みたい。英語記事は「全文翻訳（一文一文の忠実訳）＋日本語要約」を求める。現行スキーマは「公式 docs は JA なので翻訳工程なし」だったため、その例外として記事翻訳を追加（§7 共進化）。
- 確定方針：①新カテゴリ `wiki/articles/`（要約）＋ `wiki/articles/translations/`（全文翻訳・英語記事のみ）。`sources↔raw/docs` ミラーは無傷。②記事は二次資料——concept/component 本文を上書きせず引用リンクで接続（concept の関連セクションに「ブログ告知 → 記事」を 1 行）。
- 更新ファイル：
  - `CLAUDE.md`：§1 ディレクトリ（articles/・translations/）・命名（記事 slug＝blog URL 末尾・1:1 ミラー・author はプレーン文字列）・言語ポリシー（英語記事のみ全文翻訳の例外）、§2 frontmatter（`articles/*.md`・`articles/translations/*.md`）、§3 ingest 記述、§4 例外注記、§5 index に Articles セクション。
  - `.claude/skills/ingest/SKILL.md`：「記事の取り込み（raw/articles/）」節を追加（入口・言語判定・全文翻訳→要約→引用接続・移送なし）。
  - `.claude/skills/lint/SKILL.md`：`wiki/articles ↔ raw/articles` ミラー検査と「記事＝二次資料整合（concept 本文を上書きしていないか）」チェックを追加。

## [2026-06-15] ingest | articles 例 2 件（英語ブログ・全文翻訳＋要約）

- 取り込み（raw/articles/・openclaw.ai/blog の英語記事。移送なし＝原典はそのまま）:
  - https://openclaw.ai/blog/safer-than-yolo-auto-mode-for-exec-approvals
  - https://openclaw.ai/blog/openclaw-nvidia-skill-security
- 作成（articles, 2）: [[articles/safer-than-yolo-auto-mode-for-exec-approvals]], [[articles/openclaw-nvidia-skill-security]]
- 作成（translations, 2）: [[articles/translations/safer-than-yolo-auto-mode-for-exec-approvals]], [[articles/translations/openclaw-nvidia-skill-security]]
- 引用リンク追記（concept 本文は不変）: [[concepts/exec]]（`auto` モード告知）, [[components/clawhub]]（SkillSpector/ClawScan）, [[concepts/threat-model]]（T-PERSIST-001 対策・exec 承認バイパス対策）。[[index]] に Articles セクション。
- メモ:
  - 全文翻訳は §4（機械的まとめ禁止）の例外＝**忠実訳**。見出し・コードブロック・リンク URL・画像参照・表を保持し、要約/省略/意訳をしない。要約ページのみ §4 を適用。
  - **新機能（`tools.exec.mode: "auto"`・`tools.exec.reviewer.model`・Skill Card・SkillSpector・ClawScan・clawhub-security-signals データセット）はブログ時点の情報**として記事ページに置き、公式 docs ベースの concept 本文は書き換えていない。
  - author（Vincent Koc 等）は frontmatter にプレーン文字列で記録（人物 wikilink なし）。NVIDIA 記事の画像（PNG 図）は翻訳内で原 URL の参照を保持（ローカル保存はしていない）。

## [2026-06-15] query | 社内講義「アーキテクチャ編」6 記事

- 問い: OpenClaw に詳しくない社内メンバー向けに「わからないが危険」の先入観を払拭する、アーキテクチャ起点の講義を作成。
- 作成: [[lecture-architecture-00-index]], [[lecture-architecture-01-what-is-openclaw]], [[lecture-architecture-02-gateway-hub-spoke]], [[lecture-architecture-03-message-flow]], [[lecture-architecture-04-why-not-dangerous]], [[lecture-architecture-05-faq]]（すべて `wiki/questions/`）。
- 更新: [[index]]（Questions に講義シリーズを追記）。
- 参照（synthesis 元・web 検索なし）: [[overview]], [[concepts/architecture]], [[sources/concepts/architecture]], [[components/gateway]]/[[components/node]]/[[components/control-ui]]/[[components/webchat]]/[[components/cli]], [[concepts/channel-routing]], [[concepts/model-providers]], [[concepts/agent-loop]], [[concepts/messages]], [[concepts/session]], [[concepts/queue]], [[concepts/agent]], [[concepts/agent-runtimes]], [[concepts/security]], [[concepts/authentication]], [[concepts/pairing]], [[concepts/sandboxing]], [[concepts/exec]], [[concepts/threat-model]], [[concepts/secrets]], [[concepts/groups]], [[concepts/local-models]], [[concepts/remote-access]], [[sources/gateway/protocol]], [[articles/safer-than-yolo-auto-mode-for-exec-approvals]], [[articles/openclaw-nvidia-skill-security]]。
- メモ:
  - 構成: 01–03 で「わからない」を解消（形＝ハブ＆スポーク／メッセージフロー）→ 04 で「危険」を 4 柱で構造的に否定（セルフホスト・認証3層・ツール実行ゲート・脅威モデル）→ 05 で残る誤解を Q&A で払拭。
  - 受講者＝技術/非技術 混在のため、各章に身近なアナロジー（執事・中央受付・折り返し電話）＋略称初出展開（WS/DM/LLM 等）を付与（CLAUDE.md §4）。
  - 図: 02＝ハブ＆スポーク flowchart、03＝受信→返信 sequenceDiagram、04＝4 柱 flowchart＋認証3層/ツール3制御の表。
  - 二次資料（記事）は柱④の「継続強化」例として cite-only リンク。concept 等の本文は未変更（query は読むだけ）。新規 dangling ゼロ。

## [2026-06-15] ingest | articles | コンテキストエンジニアリング（Manus）

- 取り込み: `raw/articles/AIエージェントのためのコンテキストエンジニアリング：Manus構築から得た教訓.md`（元: https://manus.im/ja/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus ）
- 作成: [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]]（日本語要約・解説）。
- 接続（cite-only・本文不変）: [[concepts/context]], [[concepts/memory]], [[concepts/compaction]], [[concepts/context-engine]] の「関連」節に各 1 行の引用リンクを追加。記事側からは [[concepts/system-prompt]] / [[concepts/agent]] / [[concepts/active-memory]] / [[concepts/agent-loop]] / [[concepts/retry]] / [[concepts/agent-workspace]] / [[concepts/progress-drafts]] / [[concepts/model-providers]] へも接続。
- 更新: [[index]]（Articles に 1 行）。
- メモ:
  - **日本語記事のため全文翻訳なし**（保持ファイルが英語原文の公式 ja 訳）。`wiki/articles/translations/` には作成せず、要約ページのみ。
  - **別プロダクト（Manus／現 Meta）由来の外部二次資料**。OpenClaw 公式仕様ではなく、context/memory 系設計の「なぜ」を照らす一般原則として接続。concept 本文は書き換えていない。
  - author（Yichao "Peak" Ji）は frontmatter にプレーン文字列で記録（人物 wikilink なし）。published は原文英語版の 2025 年（正確日は本文で注記）。
  - 記事中の図（cloudfront PNG）は要約に取り込まず。原典 immutable・移送なし。

## [2026-06-15] ingest | articles | コンテキストエンジニアリング：Session による短期メモリ管理（OpenAI Cookbook）

- 取り込み: `raw/articles/Context Engineering - Short-Term Memory Management with Sessions.md`（元: https://developers.openai.com/cookbook/examples/agents_sdk/session_memory ）
- 作成: [[articles/context-engineering-session-memory]]（日本語要約・解説）＋ [[articles/translations/context-engineering-session-memory]]（**英語記事のため全文翻訳**：一文ずつの忠実訳、全コードブロック・表・画像参照を保持）。
- 接続（cite-only・本文不変）: [[concepts/session]], [[concepts/session-pruning]], [[concepts/compaction]], [[concepts/context]] の「関連」節に各 1 行の引用リンクを追加。記事側からは [[concepts/memory]] / [[concepts/context-engine]] / [[concepts/agent-loop]] / [[concepts/qa-automation]] / [[concepts/observability]] / [[concepts/model-providers]] / [[components/gateway]] へも接続。対をなす二次資料 [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]] と相互リンク。
- 更新: [[index]]（Articles に 1 行・全文翻訳リンク併記）。
- メモ:
  - **英語記事のため翻訳あり**（前回の Manus 記事は日本語版だったため翻訳なし、という差分）。翻訳は §4 の例外＝忠実訳：見出し・コード（Python/text、未訳）・リンク URL・表・画像参照を保持。
  - **別エコシステム（OpenAI Agents SDK）由来の外部二次資料**。OpenClaw 公式仕様ではなく、session/compaction/pruning の設計の「なぜ」を照らす一般原則として接続。concept 本文は書き換えていない。
  - 強い対応：記事「トリミング（ターン単位破棄）」↔ [[concepts/session-pruning]]（ただし OpenClaw はツール結果のみ＝粒度差を明記）、記事「要約（合成ペア＋直近 N 保持）」↔ [[concepts/compaction]]（ほぼ同型）。
  - slug は URL 末尾 `session_memory` を識別性のため `context-engineering-session-memory` に拡張。author は OpenAI（プレーン文字列・人物 wikilink なし）。画像 2 点は翻訳内で参照リンクのみ保持（ローカル保存せず）。原典 immutable・移送なし。

## [2026-06-15] ingest | articles | パーソナライゼーションのためのコンテキストエンジニアリング：長期メモリノートによる状態管理（OpenAI Cookbook）

- 取り込み: `raw/articles/Context Engineering for Personalization - State Management with Long-Term Memory Notes.md`（元: https://developers.openai.com/cookbook/examples/agents_sdk/context_personalization ）
- 作成: [[articles/context-engineering-personalization]]（日本語要約・解説）＋ [[articles/translations/context-engineering-personalization]]（**英語記事のため全文翻訳**：一文ずつの忠実訳、全コードブロック・JSON・表・リンクを保持）。
- 接続（cite-only・本文不変）: [[concepts/dreaming]], [[concepts/memory]], [[concepts/active-memory]], [[concepts/system-prompt]], [[concepts/security]] の「関連」節に各 1 行の引用リンクを追加。記事側からは [[concepts/commitments]] / [[concepts/memory-search]] / [[concepts/threat-model]] / [[concepts/secrets]] / [[concepts/session-pruning]] / [[concepts/context]] / [[concepts/qa-automation]] / [[concepts/observability]] へも接続。
- 更新: [[index]]（Articles に 1 行・全文翻訳リンク併記）。
- メモ:
  - **英語記事のため翻訳あり**。前編 [[articles/context-engineering-session-memory]]（短期メモリ）の**続編**＝長期メモリ。`TrimmingSession` を再利用しており相互リンクで接続。
  - **別エコシステム（OpenAI Agents SDK）由来の外部二次資料**。OpenClaw 公式仕様ではない。memory/dreaming 系設計の「なぜ」を照らす一般原則として接続。concept 本文は書き換えていない。
  - 最強の対応：記事「統合（consolidation＝重複排除・競合解決・忘却→昇格）」↔ [[concepts/dreaming]]（短期シグナル→`MEMORY.md` 昇格）。注入↔ [[concepts/system-prompt]]/[[concepts/active-memory]]、ガードレール↔ [[concepts/security]]/[[concepts/threat-model]]/[[concepts/secrets]]。
  - slug は URL 末尾 `context_personalization` を識別性のため `context-engineering-personalization` に拡張。author は OpenAI（プレーン文字列・人物 wikilink なし）。画像なし。原典 immutable・移送なし。

## [2026-06-15] ingest | articles | AI エージェントのための効果的なコンテキストエンジニアリング（Anthropic）

- 取り込み: `raw/articles/Effective context engineering for AI agents.md`（元: https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents ）
- 作成: [[articles/effective-context-engineering-for-ai-agents]]（日本語要約・解説）＋ [[articles/translations/effective-context-engineering-for-ai-agents]]（**英語記事のため全文翻訳**：一文ずつの忠実訳、見出し・リンク・図参照を保持。コードブロックは原文に無し）。
- 接続（cite-only・本文不変）: [[concepts/context]], [[concepts/compaction]], [[concepts/session-pruning]], [[concepts/memory]], [[concepts/multi-agent]], [[concepts/system-prompt]] の「関連」節に各 1 行の引用リンクを追加。記事側からは [[concepts/memory-search]] / [[concepts/agent-workspace]] / [[concepts/active-memory]] / [[concepts/progress-drafts]] / [[concepts/exec]] / [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]] / [[concepts/agent]] / [[providers/anthropic]] へも接続。
- 更新: [[index]]（Articles に 1 行・全文翻訳リンク併記）。
- メモ:
  - **Anthropic（Claude 開発元＝OpenClaw の主要プロバイダー [[providers/anthropic]]）の総論記事**。これまでの 3 本（Manus／session-memory／personalization）の**概念的土台（anchor）**として相互リンク。OpenClaw 公式仕様ではないが context/memory 設計の上流思想。
  - 最強の対応：context rot/注意の予算↔[[concepts/context]]、Compaction↔[[concepts/compaction]]、ツール結果クリア↔[[concepts/session-pruning]]、構造化ノート取り/memory tool↔[[concepts/memory]]、サブエージェント↔[[concepts/multi-agent]]、right altitude↔[[concepts/system-prompt]]。記事の「CLAUDE.md を前もって投入」は OpenClaw のブートストラップ注入（system-prompt）に対応。
  - 英語記事のため翻訳あり。**コードブロックは原文に存在せず**（散文＋図 2 点＋多数のリンク）。図はローカル保存せず翻訳内に参照リンクのみ。
  - slug は URL 末尾 `effective-context-engineering-for-ai-agents` をそのまま採用。author は Anthropic Applied AI チーム（Prithvi Rajasekaran 他、プレーン文字列・人物 wikilink なし）。原典 immutable・移送なし。

## [2026-06-15] ingest | articles | 長時間実行エージェントのための効果的なハーネス（Anthropic）

- 取り込み: `raw/articles/Effective harnesses for long-running agents.md`（元: https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents ）
- 作成: [[articles/effective-harnesses-for-long-running-agents]]（日本語要約・解説）＋ [[articles/translations/effective-harnesses-for-long-running-agents]]（**英語記事のため全文翻訳**：一文ずつの忠実訳、コードブロック・表・図参照・脚注を保持）。
- 接続（cite-only・本文不変）: [[concepts/agent-workspace]], [[concepts/memory]], [[concepts/compaction]], [[concepts/session]], [[concepts/multi-agent]] の「関連」節に各 1 行の引用リンクを追加。記事側からは [[concepts/system-prompt]] / [[concepts/exec]] / [[concepts/media-understanding]] / [[concepts/skills]] / [[concepts/agent-loop]] / [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]] / [[providers/anthropic]] へも接続。
- 更新: [[index]]（Articles に 1 行・全文翻訳リンク併記）。
- メモ:
  - **Anthropic の実践記事**で、姉妹記事 [[articles/effective-context-engineering-for-ai-agents]]（総論）の「長時間軸 3 技法」を具体ハーネスとして実装したもの。相互リンク済み。
  - 最強の対応：進捗ファイル＋git 履歴の橋渡し↔[[concepts/agent-workspace]]（private git バックアップ推奨）＋[[concepts/memory]]。離散セッション↔[[concepts/session]]。「Compaction だけでは不十分」↔[[concepts/compaction]] の限界。Future work の専門エージェント↔[[concepts/multi-agent]]。
  - **粒度の注意を要約に明記**：(1) 記事の initializer/coding は脚注いわく同一ハーネス・初期プロンプト違いで厳密にはマルチエージェントでない。(2) 記事の「テスト/自己検証」は [[concepts/exec]]/[[concepts/media-understanding]]/MCP に対応し、OpenClaw の [[concepts/qa-automation]]（メンテナー向け OpenClaw 自身の QA）とは別物なので qa-automation には cite せず本文で区別。(3) 記事の「進捗ファイル」は永続ノート＝memory/agent-workspace であり、[[concepts/progress-drafts]]（ストリーミング UX）とは別物なので cite せず。
  - 英語記事のため翻訳あり。コードフェンス 4（=2 ブロック）・表 1・図 1（GIF、ローカル保存せず参照のみ）。author は Justin Young（プレーン文字列・人物 wikilink なし）。原典 immutable・移送なし。

## [2026-06-16] ingest | ブラウザー制御（tools/browser-* 4 ページ）＋ components/browser 起こし

- 取り込み:
  - `raw/docs/tools/browser-control.md`（元: https://docs.openclaw.ai/ja-JP/tools/browser-control ）
  - `raw/docs/tools/browser-login.md`（元: https://docs.openclaw.ai/ja-JP/tools/browser-login ）
  - `raw/docs/tools/browser-linux-troubleshooting.md`（元: https://docs.openclaw.ai/ja-JP/tools/browser-linux-troubleshooting ）
  - `raw/docs/tools/browser-wsl2-windows-remote-cdp-troubleshooting.md`（元: https://docs.openclaw.ai/ja-JP/tools/browser-wsl2-windows-remote-cdp-troubleshooting ）
- 作成: [[components/browser]]（新規ランドマーク構成要素）、[[sources/tools/browser-control]]、[[sources/tools/browser-login]]、[[sources/tools/browser-linux-troubleshooting]]、[[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]]。
- 更新: [[sources/tools/browser]]（frontmatter related に components/browser 追加・位置づけ/関連にハブとサブページ）、[[concepts/session-tool]]（関連に components/browser を逆リンク＝「読む web-search／操作する browser」）、[[overview]]（構成要素一覧に Browser・現状にブラウザー制御群）、[[index]]（Components に browser・Sources/tools に 4 ページ）。
- メモ:
  - **構成要素昇格の判断**：ブラウザー制御は loopback 制御サービス＋`browser` Plugin＋CLI＋HTTP API を持つ具体サブシステムで、docs が browser＋4 サブページの計 5 ページに達した。CLAUDE.md §1「言及が増えたら独立させる」に従い [[sources/tools/browser]]（既存）をハブから外し、新規 [[components/browser]] を俯瞰ハブに据えた（source 5 本は components/browser に集約）。
  - 5 信頼境界の観点：ブラウザー制御は SSRF 防御（[[sources/security/network-proxy]]）・任意 JS（`evaluateEnabled`）・ログイン済みセッション機密の面でオペレーターアクセス相当（[[concepts/security]]/[[concepts/threat-model]]）。要約・component 本文に明記。
  - プロファイル 3 種（openclaw 隔離 / user=existing-session・Chrome MCP / リモート CDP）と、Linux の snap Chromium 問題・WSL2 分割ホストの階層別切り分けが実務の肝。
  - 移送: Clippings/ の 4 ファイルを raw/docs/tools/ の各スラグへ移動・改名（docs URL ツリー再現）。

## [2026-06-16] query | OpenClaw の強み・ユースケース・他エージェント比較

- 問い: OpenClaw の優れた点・ユースケースと、Claude Code / LangGraph で構築したエージェント / Strands Agents との比較。
- 作成: [[openclaw-vs-agent-frameworks]]（`wiki/questions/`）。
- 参照: [[overview]], [[concepts/architecture]], [[concepts/security]], [[concepts/channel-routing]], [[concepts/model-providers]], [[concepts/agent-runtimes]], [[concepts/acp]], [[concepts/memory]], [[concepts/automation]], [[concepts/multi-agent]], [[components/node]], [[components/plugin-system]] ほか。
- 更新: [[index]]（Questions に 1 行）。
- メモ:
  - 回答の骨＝「レイヤーが違う」：OpenClaw＝立てて話しかける自己ホスト型ゲートウェイ製品／Claude Code＝コーディング CLI（ACP で内包可）／LangGraph＝構築フレームワーク／Strands＝AWS の構築 SDK。比較表（4 主体 × 11 観点）＋補完関係＋選択ガイド。
  - **出典の透明性**：OpenClaw 側は全主張に wiki 引用。外部 3 者は wiki 外＝一般知識（カットオフ 2026-01）に基づく旨を明示し、Strands は WebSearch で軽く裏取り（AWS OSS・model-driven・Bedrock/Anthropic/OpenAI 対応・可観測性標準を確認）。
  - concept/overview 本文は不変（query は読むだけ）。新規 dangling ゼロ（後続の検証で確認）。
