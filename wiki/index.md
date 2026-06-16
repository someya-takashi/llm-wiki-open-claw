# Index — OpenClaw LLM Wiki カタログ

全ページの一覧。1 行 = 1 ページ（`- [[<slug>]] — <一行説明>`）。ingest / query で新規ページを作るたびに更新する。

## Overview

- [[overview]] — OpenClaw 全体像（何であるか・Gateway 中心のアーキテクチャ・主要カテゴリへの入口）

## Sources

ドキュメントのセクション階層（`raw/docs/<section>/...` ＝ `https://docs.openclaw.ai/ja-JP/<section>/...`）をそのまま反映する。

### concepts
- [[sources/concepts/architecture]] — Gateway 中心の WS トポロジー・接続ライフサイクル・ペアリング
- [[sources/concepts/agent]] — 単一エージェントプロセスの契約（ワークスペース/ブートストラップ/セッション）
- [[sources/concepts/agent-runtimes]] — 実行バックエンドの層（pi/codex/claude-cli）と選択・所有権
- [[sources/concepts/agent-loop]] — 権威ある実行サイクル（受付→推論→ツール→ストリーム→永続化）
- [[sources/concepts/agent-workspace]] — エージェントのホーム（唯一の cwd・非公開メモリ・標準ファイル群）
- [[sources/concepts/soul]] — `SOUL.md` でエージェントの声（トーン/意見/簡潔さ）を作る
- [[sources/concepts/system-prompt]] — 実行ごとに組み立てる自前システムプロンプトとキャッシュ境界
- [[sources/concepts/context]] — 1 実行でモデルへ送る全体とコンテキストウィンドウ
- [[sources/concepts/context-engine]] — コンテキスト組み立て・Compaction の差し替え可能スロット
- [[sources/concepts/oauth]] — モデルのサブスク認証（PKCE）とトークンシンク・複数アカウント
- [[sources/concepts/session]] — 会話のセッション化・DM 分離・ライフサイクル・保存
- [[sources/concepts/session-pruning]] — LLM 呼び出し前に古いツール結果を刈り込む（キャッシュ最適化）
- [[sources/concepts/session-tool]] — セッション操作・サブエージェント生成のエージェントツール
- [[sources/concepts/channel-docking]] — 返信の届け先だけを別チャネルへ転送（`/dock_*`）
- [[sources/concepts/experimental-features]] — オプトインのプレビューフラグ（ローカルモデル リーンモード等）
- [[sources/concepts/qa-e2e-automation]] — メンテナー向け QA スタック（ライブトランスポート契約）
- [[sources/concepts/qa-matrix]] — Matrix のライブ QA レーン（使い捨て homeserver）
- [[sources/concepts/memory]] — Markdown ファイルによるメモリの全体像（ファイル/ツール/バックエンド）
- [[sources/concepts/memory-builtin]] — 既定の SQLite メモリエンジン（FTS5＋ベクトル）
- [[sources/concepts/memory-qmd]] — 再ランキング付きローカル優先サイドカー QMD
- [[sources/concepts/memory-honcho]] — 自動ユーザーモデリングの AI ネイティブメモリ（Plugin）
- [[sources/concepts/memory-search]] — ハイブリッド検索（ベクトル＋BM25）とプロバイダー
- [[sources/concepts/active-memory]] — 応答前に先回りで想起する Plugin サブエージェント
- [[sources/concepts/dreaming]] — 短期シグナルを長期メモリへ昇格するバックグラウンド統合
- [[sources/concepts/commitments]] — 会話から推論する短期フォローアップメモリ
- [[sources/concepts/compaction]] — 古い会話を要約してウィンドウを空ける
- [[sources/concepts/multi-agent]] — 1 Gateway で複数エージェントを分離・ルーティング（バインディング）
- [[sources/concepts/delegate-architecture]] — 組織向け「代理エージェント」の権限ティアと強化
- [[sources/concepts/parallel-specialist-lanes]] — 並列ワークをレーン契約で設計する手法
- [[sources/concepts/presence]] — Gateway と接続クライアントの軽量な状態ビュー
- [[sources/concepts/messages]] — 受信→返信のメッセージ処理パイプライン（重複排除/デバウンス/サイレント返信）
- [[sources/concepts/queue]] — インバウンド実行のレーン対応 FIFO キューとキューモード
- [[sources/concepts/queue-steering]] — ステアリングのランタイム別配信タイミング（Pi/Codex）
- [[sources/concepts/streaming]] — ブロック/プレビューの 2 層ストリーミングとチャンク化
- [[sources/concepts/progress-drafts]] — 長いターンを 1 つの作業中下書きで見せる（progress モード）
- [[sources/concepts/retry]] — HTTP リクエスト単位の再試行ポリシーとフェイルオーバー
- [[sources/concepts/message-lifecycle-refactor]] — 送受信ライフサイクルの将来設計（RFC・内部）
- [[sources/concepts/model-providers]] — 全モデルプロバイダーの設定リファレンス（組み込み＋カスタム）
- [[sources/concepts/model-failover]] — 認証ローテーション＋モデルフォールバック＋クールダウン
- [[sources/concepts/models]] — モデル選択ルールと `openclaw models` CLI・レジストリ

### start
- [[sources/start/bootstrapping]] — 初回実行でワークスペースと ID を準備する手順

### gateway
- [[sources/gateway/gateway]] — Gateway 運用手順書（起動・監視・サービス・失敗シグネチャ）
- [[sources/gateway/configuration]] — 設定のタスク指向ガイド（JSON5・検証・ホットリロード・$include）
- [[sources/gateway/configuration-reference]] — 全フィールドの網羅リファレンス（トップレベル地図）
- [[sources/gateway/configuration-examples]] — コピペ可能な設定例集（用途別パターン）
- [[sources/gateway/config-agents]] — エージェント設定（agents/session/messages/sandbox/multi-agent）
- [[sources/gateway/config-channels]] — チャネル設定（DM/グループ・メンションゲート・複数アカウント）
- [[sources/gateway/config-tools]] — ツール設定とカスタムプロバイダー（base URL）
- [[sources/gateway/authentication]] — モデルプロバイダー認証（API キー/OAuth/Claude CLI/auth-profiles）
- [[sources/gateway/trusted-proxy-auth]] — リバースプロキシに認証を委任する接続認証モード
- [[sources/gateway/secrets]] — SecretRef による秘密管理（env/file/exec・スナップショット）
- [[sources/gateway/secrets-plan-contract]] — `secrets apply` の厳密なターゲット/パス契約
- [[sources/gateway/heartbeat]] — メインセッションで定期実行するエージェントターン
- [[sources/gateway/health]] — チャネル接続性と Gateway 健全性の確認（status/health）
- [[sources/gateway/doctor]] — 設定/状態の修復・移行ツール（doctor）
- [[sources/gateway/troubleshooting]] — 症状起点の詳細ランブック
- [[sources/gateway/diagnostics]] — 共有可能なサポート診断 zip と安定性レコーダー
- [[sources/gateway/logging]] — Gateway 内部の WS ログ/サブシステム整形
- [[sources/gateway/prometheus]] — Prometheus メトリクス（プル型エクスポート）
- [[sources/gateway/opentelemetry]] — OpenTelemetry エクスポート（OTLP プッシュ・3 信号）
- [[sources/gateway/gateway-lock]] — 1 ホスト 1 Gateway を保証する起動ロック機構
- [[sources/gateway/multiple-gateways]] — 別プロファイル/ポートで複数 Gateway を分離運用（レスキューボット）
- [[sources/gateway/background-process]] — `exec`/`process` ツールによる長時間バックグラウンド実行
- [[sources/gateway/security]] — セキュリティの脅威モデルと強化ベースライン（ハブ）
- [[sources/gateway/sandboxing]] — ツール実行のサンドボックス分離（モード/スコープ/バックエンド）
- [[sources/gateway/sandbox-vs-tool-policy-vs-elevated]] — サンドボックス/ツールポリシー/昇格の 3 制御の違い
- [[sources/gateway/openshell]] — マネージドリモートサンドボックスバックエンド（mirror/remote）
- [[sources/gateway/operator-scopes]] — 認証後の権限（operator.read/write/admin 等）
- [[sources/gateway/protocol]] — Gateway WS プロトコルの形式的な契約（ハンドシェイク/ロール/RPC/イベント）
- [[sources/gateway/pairing]] — Gateway 管理のノード/デバイスペアリング（承認とトークン発行）
- [[sources/gateway/discovery]] — 検出とトランスポート（Direct WS / SSH・3 つの検出入力）
- [[sources/gateway/bonjour]] — `_openclaw-gw._tcp` ビーコン（mDNS/DNS-SD・TXT キー・TLS pinning）
- [[sources/gateway/bridge-protocol]] — レガシー TCP ブリッジ（削除済み・履歴）
- [[sources/gateway/openai-http-api]] — OpenAI 互換 `/v1/chat/completions`（エージェント優先モデル契約）
- [[sources/gateway/openresponses-http-api]] — OpenResponses `/v1/responses`（項目入力・マルチモーダル）
- [[sources/gateway/tools-invoke-http-api]] — 単一ツール直接呼び出し `/tools/invoke`（ハード拒否リスト）
- [[sources/gateway/local-models]] — 自前ホスト LLM の登録・推奨スタック・互換シム
- [[sources/gateway/local-model-services]] — `localService` でローカルサーバーをオンデマンド起動
- [[sources/gateway/cli-backends]] — ローカル AI CLI のテキスト専用フォールバック（claude-cli/codex）
- [[sources/gateway/remote]] — リモートアクセス（SSH トンネル・Tailscale・bind モード・認証情報優先順位）
- [[sources/gateway/remote-gateway-readme]] — OpenClaw.app の SSH トンネル手順（remote に統合済み）
- [[sources/gateway/tailscale]] — Tailscale Serve/Funnel と ID ヘッダー認証
- [[sources/gateway/security/audit-checks]] — `openclaw security audit` の checkId カタログ
- [[sources/gateway/security/secure-file-operations]] — `@openclaw/fs-safe` のファイル操作ガードレール

### nodes
- [[sources/nodes/nodes]] — ノードの総覧（コマンドサーフェス・ノードホスト・system.run・Mac Node）
- [[sources/nodes/camera]] — カメラ写真/動画クリップ・画面録画（フォアグラウンド必須）
- [[sources/nodes/audio]] — ボイスメモの文字起こしとグループのメンション preflight
- [[sources/nodes/images]] — メディア送受信のサイズ規則と受信テンプレ変数
- [[sources/nodes/media-understanding]] — `tools.media` による受信メディアのテキスト要約
- [[sources/nodes/talk]] — 継続的な音声会話ループ（realtime/stt-tts/transcription）
- [[sources/nodes/voicewake]] — Gateway 所有のウェイクワードとルーティング
- [[sources/nodes/location-command]] — `location.get`（既定オフ・権限セレクター）
- [[sources/nodes/troubleshooting]] — ノードの 3 ゲート切り分けとエラーコード

### tools
- [[sources/tools/tools]] — ツール/Skills/Plugins の選び分け（capabilities 概観）
- [[sources/tools/plugin]] — Plugin のインストール・設定・管理・登録 API（ネイティブ/バンドル）
- [[sources/tools/skills]] — `SKILL.md` 指示パック（読み込み/優先順位/ゲート/許可リスト）
- [[sources/tools/creating-skills]] — カスタムワークスペース Skill の作成とテスト
- [[sources/tools/skills-config]] — `skills.*` / `agents.*.skills` 設定スキーマ
- [[sources/tools/slash-commands]] — `/...` コマンド・ディレクティブ・ネイティブ/テキスト・オーナー専用
- [[sources/tools/browser]] — エージェント専用ブラウザー（Chrome/CDP・openclaw/user プロファイル）
- [[sources/tools/browser-control]] — ブラウザー制御 HTTP API・`openclaw browser` CLI・スナップショット/ref・デバッグ
- [[sources/tools/browser-login]] — 手動ログイン推奨と X/Twitter フロー（サンドボックス × ホスト制御）
- [[sources/tools/browser-linux-troubleshooting]] — Linux の CDP 起動失敗（snap Chromium 問題）
- [[sources/tools/browser-wsl2-windows-remote-cdp-troubleshooting]] — WSL2 Gateway＋Windows Chrome のリモート CDP を階層別に切り分け
- [[sources/tools/web]] — Web 検索の概要（web_search/x_search・12 プロバイダー・自動検出）
- [[sources/tools/web-fetch]] — `web_fetch`（URL 取得・Readability・SSRF・Firecrawl フォールバック）
- [[sources/tools/brave-search]] — Brave（構造化・llm-context・無料枠）
- [[sources/tools/duckduckgo-search]] — DuckDuckGo（キー不要フォールバック）
- [[sources/tools/exa-search]] — Exa（ニューラル＋キーワード・コンテンツ抽出）
- [[sources/tools/firecrawl]] — Firecrawl（検索＋スクレイピング・web_fetch フォールバック）
- [[sources/tools/gemini-search]] — Gemini（Google グラウンディング・引用付き AI 合成）
- [[sources/tools/grok-search]] — Grok（xAI グラウンディング・x_search 共有）
- [[sources/tools/kimi-search]] — Kimi（Moonshot・根拠なしは失敗）
- [[sources/tools/minimax-search]] — MiniMax（構造化・リージョン）
- [[sources/tools/ollama-search]] — Ollama（キー不要・ローカルホスト）
- [[sources/tools/perplexity-search]] — Perplexity（ドメインフィルタ・Sonar/OpenRouter）
- [[sources/tools/searxng-search]] — SearXNG（セルフホスト型メタ検索）
- [[sources/tools/tavily]] — Tavily（検索深度・tavily_extract）
- [[sources/tools/subagents]] — サブエージェント（`sessions_spawn`・バックグラウンド委任）
- [[sources/tools/agent-send]] — `openclaw agent`（CLI から単一ターン実行）
- [[sources/tools/multi-agent-sandbox-tools]] — エージェントごとのサンドボックス/ツール上書き
- [[sources/tools/steer]] — `/steer`（実行中ターンへのガイダンス注入）
- [[sources/tools/acp-agents]] — ACP エージェント（外部ハーネスのバインド済みセッション）
- [[sources/tools/acp-agents-setup]] — ACP の設定（acpx・MCP ブリッジ・権限）
- [[sources/tools/exec]] — シェル実行ツール（exec/process・host=auto/gateway/node/sandbox）
- [[sources/tools/exec-approvals]] — 実ホスト実行の承認ガードレール（ポリシー＋許可リスト＋承認）
- [[sources/tools/exec-approvals-advanced]] — safe bins・インタプリタ束縛・承認転送
- [[sources/tools/elevated]] — 昇格モード（サンドボックス外し・`/elevated`）
- [[sources/tools/code-execution]] — `code_execution`（xAI Responses API のリモート Python）
- [[sources/tools/apply-patch]] — 構造化パッチでの複数ファイル編集
- [[sources/tools/diffs]] — 変更を読み取り専用 diff 成果物に（Plugin ツール）
- [[sources/tools/tool-search]] — 大規模ツールカタログの検索呼び出し（PI 実験的）
- [[sources/tools/tokenjuice]] — exec/bash のツール結果を圧縮（Plugin）
- [[sources/tools/loop-detection]] — 反復ツール呼び出しのガードレール
- [[sources/tools/lobster]] — 承認チェックポイント付きの決定的ワークフローシェル
- [[sources/tools/llm-task]] — JSON 構造化出力の単一 LLM ステップ（Plugin）
- [[sources/tools/btw]] — 履歴を変えない横道質問（`/btw`・`/side`）
- [[sources/tools/trajectory]] — セッションのフライトレコーダーと `/export-trajectory`
- [[sources/tools/thinking]] — 思考レベル（`/think` reasoning effort）
- [[sources/tools/media-overview]] — メディア機能の地図（生成/理解/音声）
- [[sources/tools/image-generation]] — `image_generate`（画像の生成/編集）
- [[sources/tools/music-generation]] — `music_generate`（音楽/音声・バックグラウンドタスク）
- [[sources/tools/pdf]] — PDF を解析してテキスト化
- [[sources/tools/reactions]] — `message` ツールの絵文字リアクション
- [[sources/tools/tts]] — テキスト読み上げ（14 プロバイダー・ペルソナ・チャネル別出力）
- [[sources/tools/video-generation]] — `video_generate`（非同期タスク・16 プロバイダー）

### automation
- [[sources/automation/automation]] — 自動化の選び分け（Cron/Heartbeat/Tasks/Hooks/…）
- [[sources/automation/cron-jobs]] — スケジュールされたタスク（at/every/cron・配信・Gmail）
- [[sources/automation/tasks]] — バックグラウンドタスク台帳（ライフサイクル/通知/監査）
- [[sources/automation/taskflow]] — Task Flow（複数ステップの耐久オーケストレーション）
- [[sources/automation/hooks]] — 内部フック（HOOK.md・ライフサイクル/メッセージイベント）
- [[sources/automation/standing-orders]] — 全セッションに注入される常設指示

### plugins
- [[sources/plugins/manage-plugins]] — インストール/更新/アンインストール/公開のコマンド集
- [[sources/plugins/bundles]] — Codex/Claude/Cursor バンドルの機能マッピング（Skills/MCP/LSP）
- [[sources/plugins/community]] — サードパーティ Plugin カタログと提出ガイド
- [[sources/plugins/codex-harness]] — Codex app-server を使うエージェントランタイム Plugin
- [[sources/plugins/codex-computer-use]] — Codex ネイティブのデスクトップ制御 MCP
- [[sources/plugins/codex-native-plugins]] — 移行済みネイティブ Codex プラグイン（app-server アプリ）
- [[sources/plugins/google-meet]] — Google Meet 会議参加（文字起こし→応答→TTS）
- [[sources/plugins/voice-call]] — 電話通話（Twilio/Telnyx/Plivo・全二重リアルタイム）
- [[sources/plugins/memory-lancedb]] — LanceDB ベクトル DB の長期メモリ（Active Memory）
- [[sources/plugins/memory-wiki]] — 永続メモリを知識 Vault（wiki）へコンパイル
- [[sources/plugins/webhooks]] — 外部自動化を TaskFlow に結ぶ認証付き Webhook 入口
- [[sources/plugins/oc-path]] — `oc://` ファイルアドレス指定の `openclaw path` CLI
- [[sources/plugins/zalouser]] — Zalo 個人アカウント連携（チャネル Plugin・QR ログイン）
- [[sources/plugins/building-plugins]] — ネイティブ Plugin の構築入門（`register(api)`・提出前チェック）
- [[sources/plugins/hooks]] — Plugin ライフサイクルフック（before_tool_call / llm_input 等）
- [[sources/plugins/sdk-channel-plugins]] — チャネル Plugin の構築（DM/ペアリング/メンション）
- [[sources/plugins/sdk-provider-plugins]] — モデルプロバイダー Plugin の構築（カタログ/認証）
- [[sources/plugins/cli-backend-plugins]] — CLI バックエンド Plugin の構築（テキストフォールバック）
- [[sources/plugins/adding-capabilities]] — コアに新 capability を足すコントリビューターガイド

### security
- [[sources/security/network-proxy]] — 送信フォワードプロキシによる SSRF 強化（任意の多層防御）
- [[sources/security/threat-model-atlas]] — MITRE ATLAS 脅威モデル（5 信頼境界・攻撃カテゴリ）
- [[sources/security/formal-verification]] — TLA+/TLC によるセキュリティ不変条件の機械検証
- [[sources/security/contributing-threat-model]] — 脅威モデルへの貢献ガイド

### web
- [[sources/web/web]] — Gateway の Web サーフェス（Control UI 配信・バインドモード・セキュリティ）
- [[sources/web/control-ui]] — ブラウザー管理ダッシュボード（Vite+Lit SPA・ペアリング・設定編集・PWA）
- [[sources/web/webchat]] — Gateway 直結のネイティブチャット UI（トランスクリプト/配信モデル）
- [[sources/web/tui]] — ターミナル UI（Gateway/ローカルモード・スラッシュコマンド）

### (root) — トップレベル文書
- [[sources/auth-credential-semantics]] — 認証情報の適格性・プローブ理由コード・移植性
- [[sources/logging]] — ロギングの利用者向け概要（保存場所・読み方・レベル・秘匿化）
- [[sources/network]] — localhost/LAN/tailnet の接続・ペアリング・保護のハブ
- [[sources/prose]] — OpenProse（`.prose` マルチエージェントワークフロー DSL の Plugin）

### channels
- [[sources/channels/channels]] — 全チャットチャネルのカタログ（25+・配信注記）
- [[sources/channels/discord]] — Discord（Bot API・ギルド/DM・ネイティブコマンド・ボイス）
- [[sources/channels/slack]] — Slack（Bolt/Socket Mode・MPIM はグループ扱い）
- [[sources/channels/telegram]] — Telegram（grammY・最速セットアップ・グループ/トピック）
- [[sources/channels/whatsapp]] — WhatsApp（Baileys・QR ペアリング・単一セッション）
- [[sources/channels/channel-routing]] — 決定的な返信ルーティング・セッションキー・宛先プレフィックス
- [[sources/channels/groups]] — グループチャット挙動（メンションゲート・許可リスト・ツール制限）
- [[sources/channels/broadcast-groups]] — 1 グループに複数エージェント（parallel/sequential・実験的）
- [[sources/channels/access-groups]] — 名前付き送信者グループ（`accessGroup:<name>`）
- [[sources/channels/group-messages]] — WhatsApp 固有のグループ挙動
- [[sources/channels/pairing]] — ペアリング（DM ＋ Node の明示的アクセス承認）
- [[sources/channels/location]] — チャットで共有された位置情報の解析
- [[sources/channels/troubleshooting]] — チャネル別の診断手順
- [[sources/channels/qa-channel]] — QA 用の合成チャネルトランスポート

### providers
- [[sources/providers/providers]] — プロバイダーディレクトリ（50+ の LLM プロバイダー）
- [[sources/providers/models]] — モデルプロバイダーのクイックスタート（2 ステップ）
- [[sources/providers/anthropic]] — Anthropic（Claude・API キー/Claude CLI）
- [[sources/providers/openai]] — OpenAI（GPT/Codex・既定で Codex ランタイム）
- [[sources/providers/google]] — Google（Gemini・画像/メディア/TTS/Web 検索）
- [[sources/providers/ollama]] — Ollama（ローカル/クラウド 3 モード）
- [[sources/providers/bedrock]] — Amazon Bedrock（AWS 認証情報チェーン）
- [[sources/providers/litellm]] — LiteLLM（統合 Gateway・100+ プロバイダー）

### platforms
- [[sources/platforms/platforms]] — OS 別の実行・コンパニオンアプリ・サービス
- [[sources/platforms/macos]] — macOS メニューバーアプリ（Node＋Control UI ホスト）
- [[sources/platforms/ios]] — iOS モバイルノード（内部プレビュー）
- [[sources/platforms/android]] — Android モバイルノード（個人データコマンド・自前ビルド）
- [[sources/platforms/windows]] — Windows（ネイティブ/WSL2・スケジュールタスク）
- [[sources/platforms/linux]] — Linux（完全サポート・systemd・VPS）

### announcements
- [[sources/announcements/bluebubbles-imessage]] — BlueBubbles 削除・imsg iMessage 経路への移行

<!-- 取り込んだ docs セクションに応じて見出し（tools / cli / plugins / automation / nodes / install ...）を追加する -->

## Concepts

- [[concepts/architecture]] — Gateway 中心のハブ＆スポーク構成（俯瞰）
- [[concepts/agent]] — 単一エージェント「プロセス」の契約 ⚠️[[concepts/agent-runtimes]] と混同注意
- [[concepts/agent-runtimes]] — ターンを実行する「バックエンド層」 ⚠️[[concepts/agent]] と混同注意
- [[concepts/agent-loop]] — エージェント実行サイクルの配線
- [[concepts/agent-workspace]] — エージェントのホーム（唯一の cwd・非公開メモリ）
- [[concepts/soul]] — `SOUL.md`（人格・トーン・境界）
- [[concepts/system-prompt]] — 実行ごとに組み立てる自前システムプロンプト
- [[concepts/context]] — モデルへ送る全体（コンテキストウィンドウ）
- [[concepts/context-engine]] — コンテキスト組み立て・Compaction の差し替えスロット
- [[concepts/oauth]] — モデルのサブスク認証（OAuth/PKCE）とトークン管理
- [[concepts/session]] — 会話のセッション化・DM 分離・ライフサイクル
- [[concepts/session-pruning]] — 古いツール結果の刈り込み（プルーニング ≠ Compaction）
- [[concepts/session-tool]] — セッション操作・サブエージェント生成のツール群
- [[concepts/channel-docking]] — 返信ルートだけを別チャネルへ転送
- [[concepts/qa-automation]] — メンテナー向け QA スタックとライブトランスポート契約
- [[concepts/memory]] — Markdown ファイルによるメモリ（ハブ）と差し替え可能バックエンド
- [[concepts/memory-search]] — ハイブリッド検索（ベクトル＋BM25）
- [[concepts/active-memory]] — 応答前の先回り想起（Plugin サブエージェント）
- [[concepts/dreaming]] — 短期→長期メモリのバックグラウンド昇格
- [[concepts/commitments]] — 推論された短期フォローアップメモリ
- [[concepts/compaction]] — 会話の要約圧縮（≠ [[concepts/session-pruning]]）
- [[concepts/multi-agent]] — 1 Gateway で複数エージェントを分離・ルーティング
- [[concepts/delegate-architecture]] — 組織向け代理エージェント（権限ティア・強化）
- [[concepts/parallel-specialist-lanes]] — 並列ワークのレーン契約による設計論
- [[concepts/presence]] — Gateway と接続クライアントの状態ビュー（Instances）
- [[concepts/messages]] — 受信→返信のメッセージ処理パイプライン（ハブ）
- [[concepts/queue]] — レーン対応 FIFO キューとキューモード（steer 等。steering も統合）
- [[concepts/streaming]] — ブロック/プレビューの 2 層ストリーミング
- [[concepts/progress-drafts]] — 長いターンを単一の作業中下書きで見せる
- [[concepts/retry]] — HTTP リクエスト単位の再試行とフェイルオーバー
- [[concepts/configuration]] — `openclaw.json`（JSON5）による設定・厳格検証・ホットリロード
- [[concepts/authentication]] — 認証の 3 レイヤー（接続認証・モデル認証・デバイスペアリング）
- [[concepts/secrets]] — SecretRef による秘密の外部化（env/file/exec・スナップショット）
- [[concepts/heartbeat]] — メインセッションで定期実行する自発的エージェントターン
- [[concepts/logging]] — ファイル/コンソールログ・レベル分離・秘匿化
- [[concepts/observability]] — メトリクス/トレース/ログの外部エクスポート（Prometheus/OTel）
- [[concepts/diagnostics]] — 健全性確認・修復(doctor)・トラブルシュート・診断 zip
- [[concepts/security]] — パーソナルアシスタント信頼モデル・脅威モデル・強化
- [[concepts/sandboxing]] — ツール実行の分離（サンドボックス/ツールポリシー/昇格の 3 制御）
- [[concepts/pairing]] — ノード/デバイスを信頼集合に登録（認証 3 層の③・トークン発行）
- [[concepts/discovery]] — Gateway の発見と接続経路（Bonjour/tailnet/SSH・Direct WS）
- [[concepts/remote-access]] — 外から Gateway へ安全に到達（SSH トンネル / Tailscale / bind モード）
- [[concepts/http-api]] — OpenAI 互換 HTTP サーフェス（chat completions / responses / tools-invoke）
- [[concepts/local-models]] — 自前ホスト LLM での実行（登録・オンデマンド起動・CLI フォールバック）
- [[concepts/media-understanding]] — 受信メディア（画像/音声/動画）のテキスト要約（入力側）
- [[concepts/voice]] — 聞く/話す音声機能（Talk ループ・TTS・Voice Wake）
- [[concepts/threat-model]] — 敵対的脅威の体系化（MITRE ATLAS・5 信頼境界・形式検証）
- [[concepts/skills]] — `SKILL.md` でエージェントに「やり方」を教える指示パック
- [[concepts/slash-commands]] — `/...` でセッション/Gateway を操作する明示的な制御面
- [[concepts/exec]] — コマンド実行面（exec/process/code_execution＋承認＋昇格）
- [[concepts/automation]] — バックグラウンド作業の総称（Cron/Heartbeat/Tasks/Hooks/Standing Orders）
- [[concepts/cron]] — 時刻/間隔/cron 式での正確なスケジュール実行
- [[concepts/tasks]] — 切り離した作業のタスク台帳＋Task Flow
- [[concepts/hooks]] — ライフサイクル/メッセージイベントに反応する内部フック
- [[concepts/web-search]] — Web 検索/取得（web_search/x_search/web_fetch＋12 プロバイダー）
- [[concepts/acp]] — Agent Client Protocol（外部コーディングハーネスを丸ごと実行）
- [[concepts/channel-routing]] — チャネルルーティング（25+ チャットチャネルの入口/出口）
- [[concepts/groups]] — グループチャット挙動（メンションゲート・アクセスグループ・ブロードキャスト）
- [[concepts/model-providers]] — モデルプロバイダー（`provider/model`・50+ プロバイダー）
- [[concepts/model-failover]] — モデルフェイルオーバー（認証ローテーション＋フォールバック）

## Components

- [[components/gateway]] — ホストごと 1 つの常駐デーモン（全接続を所有）
- [[components/node]] — `role: node` で WS 接続する端末（カメラ/画面/位置を公開）
- [[components/control-ui]] — Gateway 配信のブラウザー管理ダッシュボード（Vite+Lit SPA）
- [[components/webchat]] — Gateway 直結のネイティブチャット UI（macOS/iOS SwiftUI）
- [[components/cli]] — `openclaw` コマンド群と対話的な端末 TUI（Gateway/ローカルモード）
- [[components/plugin-system]] — 拡張機構（チャネル/プロバイダー/ツール/音声…をネイティブ/バンドルで追加）
- [[components/clawhub]] — Plugin/Skill マーケットプレイス（公開・検索・モデレーション）
- [[components/browser]] — エージェントが実ブラウザーを操作するサブシステム（制御サービス＋`browser` Plugin・CDP/Playwright・プロファイル）

## Channels

- [[channels/discord]] — Discord（Bot API・ギルド/DM・ネイティブコマンド）
- [[channels/slack]] — Slack（Bolt/Socket Mode・MPIM はグループ扱い）
- [[channels/telegram]] — Telegram（grammY・最速セットアップ）
- [[channels/whatsapp]] — WhatsApp（Baileys・QR ペアリング・最も人気）
- [[channels/zalouser]] — Zalo 個人アカウント連携（非公式 `zca-js`・QR ログイン）

## Providers

- [[providers/anthropic]] — Anthropic（Claude・API キー/Claude CLI）
- [[providers/openai]] — OpenAI（GPT/Codex・既定で Codex ランタイム）
- [[providers/google]] — Google（Gemini・画像/メディア/TTS/Web 検索）
- [[providers/ollama]] — Ollama（ローカル/クラウド・[[concepts/local-models]] の代表）
- [[providers/bedrock]] — Amazon Bedrock（AWS 認証情報チェーン）
- [[providers/litellm]] — LiteLLM（統合 Gateway・100+ プロバイダー）

## Articles

非公式記事（ブログ等）の日本語要約。英語記事は全文翻訳も併記。二次資料として既存 concept へ引用リンクで接続する。

- [[articles/safer-than-yolo-auto-mode-for-exec-approvals]] — Exec 承認の第3モード `auto`（reviewer-first）の告知（→ [[concepts/exec]]）｜全文翻訳 [[articles/translations/safer-than-yolo-auto-mode-for-exec-approvals]]
- [[articles/openclaw-nvidia-skill-security]] — NVIDIA 協業の Skill Card / SkillSpector / ClawScan（→ [[components/clawhub]] / [[concepts/threat-model]]）｜全文翻訳 [[articles/translations/openclaw-nvidia-skill-security]]
- [[articles/context-engineering-for-ai-agents-lessons-from-building-manus]] — Manus 流コンテキストエンジニアリング 6 原則（外部・二次資料。→ [[concepts/context]] / [[concepts/memory]]）｜日本語記事のため翻訳なし
- [[articles/context-engineering-session-memory]] — Session による短期メモリ管理＝トリミング vs 要約（OpenAI Cookbook・外部二次資料。→ [[concepts/session-pruning]] / [[concepts/compaction]]）｜全文翻訳 [[articles/translations/context-engineering-session-memory]]
- [[articles/context-engineering-personalization]] — 長期メモリ＝状態管理（蒸留→統合→注入）でパーソナライズ（OpenAI Cookbook・外部二次資料。前編の続編。→ [[concepts/dreaming]] / [[concepts/memory]] / [[concepts/active-memory]]）｜全文翻訳 [[articles/translations/context-engineering-personalization]]
- [[articles/effective-context-engineering-for-ai-agents]] — コンテキストエンジニアリング**総論**（Anthropic。注意の予算・context rot・compaction/ノート取り/サブエージェント。上記 3 本の概念的土台。→ [[concepts/context]] / [[concepts/compaction]] / [[concepts/multi-agent]]）｜全文翻訳 [[articles/translations/effective-context-engineering-for-ai-agents]]
- [[articles/effective-harnesses-for-long-running-agents]] — 複数コンテキストウィンドウをまたぐ長時間タスクのハーネス（Anthropic。初期化＋コーディングエージェント、進捗ファイル＋git で橋渡し。→ [[concepts/agent-workspace]] / [[concepts/memory]] / [[concepts/session]]）｜全文翻訳 [[articles/translations/effective-harnesses-for-long-running-agents]]

## Questions

- 講義: アーキテクチャ編（6 章。社内向け「わからないが危険」の払拭）
  - [[lecture-architecture-00-index]] — 目次・ねらい・読み方
  - [[lecture-architecture-01-what-is-openclaw]] — OpenClaw とは何か（セルフホスト＝データ所有）
  - [[lecture-architecture-02-gateway-hub-spoke]] — 核：Gateway 中心のハブ＆スポーク（図）
  - [[lecture-architecture-03-message-flow]] — 受信から返信までの流れ（シーケンス図）
  - [[lecture-architecture-04-why-not-dangerous]] — なぜ『危険』ではないのか（安全 4 柱）
  - [[lecture-architecture-05-faq]] — よくある誤解 Q&A
- [[openclaw-vs-agent-frameworks]] — OpenClaw の強み・ユースケースと Claude Code / LangGraph / Strands Agents との比較（レイヤーの違い＋比較表）

---

## 略称リダイレクト

専用ページは作らず、対応する正式名称・ページへ読み替える。

- WS → WebSocket
- MCP → Model Context Protocol
- CLI → Command Line Interface（→ [[components/cli]]）
- TUI → Terminal User Interface（→ [[components/cli]] の対話 UI）
- SPA → Single Page Application / PWA → Progressive Web App（→ [[components/control-ui]]）
- A2UI → Agent-to-UI
- LLM → Large Language Model
- PKCE → Proof Key for Code Exchange（→ [[concepts/oauth]]）
- cwd → current working directory（→ [[concepts/agent-workspace]]）
- TTL → Time To Live（→ [[concepts/session-pruning]]）
- DM → Direct Message（→ [[concepts/session]]）
- QA → Quality Assurance（→ [[concepts/qa-automation]]）
- E2E → End-to-End / E2EE → End-to-End Encryption（→ [[sources/concepts/qa-matrix]]）
- SUT → System Under Test（→ [[concepts/qa-automation]]）
- BM25 → キーワード検索のランキング関数 / FTS → Full-Text Search（→ [[concepts/memory-search]]）
- MMR → Maximal Marginal Relevance（→ [[concepts/memory-search]]）
- GGUF → ローカルモデルの重みファイル形式（→ [[concepts/local-models]]）
- DNS-SD → DNS Service Discovery / mDNS → Multicast DNS（→ [[concepts/discovery]]）
- RCE → Remote Code Execution（→ [[concepts/http-api]] のツール拒否リスト）
- FIFO → First In, First Out（→ [[concepts/queue]]）
- RFC → Request for Comments（設計提案。→ [[sources/concepts/message-lifecycle-refactor]]）
- OTel → OpenTelemetry / OTLP → OpenTelemetry Protocol（→ [[concepts/observability]]）
- PromQL → Prometheus Query Language（→ [[concepts/observability]]）
- JSONL → JSON Lines（→ [[concepts/logging]]）
- TOCTOU → Time-Of-Check to Time-Of-Use（→ [[sources/gateway/security/secure-file-operations]]）
- DooD → Docker-out-of-Docker / CDP → Chrome DevTools Protocol（→ [[concepts/sandboxing]]）
- RBAC → Role-Based Access Control（→ [[sources/gateway/operator-scopes]]）
- TTS → Text-to-Speech / STT → Speech-to-Text（→ [[concepts/voice]] / [[concepts/media-understanding]]）
- WebRTC → Web Real-Time Communication（→ [[concepts/voice]]）
- MITRE ATLAS → Adversarial Threat Landscape for AI Systems（→ [[concepts/threat-model]]）
- TLA+ / TLC → 形式仕様記述言語とモデルチェッカー（→ [[sources/security/formal-verification]]）
- DoS → Denial of Service（→ [[concepts/threat-model]]）
- T2V / I2V / V2V → Text/Image/Video-to-Video（→ [[sources/tools/video-generation]]）
- ptt → Push-To-Talk（ボイスメモ。→ [[sources/nodes/images]]）
- LSP → Language Server Protocol（→ [[sources/plugins/bundles]]）
- PSTN / SIP → 公衆電話網 / IP 電話プロトコル（→ [[sources/plugins/voice-call]]）
- ClawHub → OpenClaw の Plugin/Skill マーケットプレイス（→ [[components/clawhub]]）
- DSL → Domain-Specific Language（→ [[sources/prose]]）
- SDK → Software Development Kit（→ [[components/plugin-system]] の Plugin SDK）
- ACP → Agent Client Protocol（→ [[concepts/acp]]）
- SSRF → Server-Side Request Forgery（→ [[concepts/web-search]] / [[concepts/security]]）
