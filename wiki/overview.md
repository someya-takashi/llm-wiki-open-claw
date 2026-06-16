---
type: overview
updated: 2026-06-14
---

# OpenClaw 全体像

> このページは OpenClaw 全体の入口（総括）。ingest が進むにつれ随時更新する。各カテゴリの詳細は `[[index]]` から辿れる。

## OpenClaw とは

**OpenClaw** は、あらゆる OS（macOS / Linux / iOS / Android / ヘッドレスサーバー）で動作する **AI エージェント用のセルフホスト型 Gateway（ゲートウェイ）** である。WhatsApp・Telegram・Slack・Discord・Signal・iMessage・WebChat などの**メッセージングサーフェス**と、Anthropic・OpenAI・Ollama などの**モデルプロバイダー**を結びつけ、ユーザーが自分のインフラ上でデータの所有権を保ったまま、複数のチャネルから同一の AI アシスタントを使えるようにする。クラウド SaaS に丸ごと預けるのではなく「自分でホストする」点が中核思想である。

## アーキテクチャの骨格

OpenClaw の中心には **単一の常駐プロセス [[components/gateway]]（Gateway）** がある。Gateway は次を担う：

- すべてのメッセージング接続（プロバイダー）を所有する。ホストごとに Gateway は 1 つ（例: WhatsApp セッションを開く唯一の場所）。
- コントロールプレーンのクライアント（[[components/control-ui]]＝ブラウザー管理 UI・[[components/webchat]]＝ネイティブチャット・[[components/cli]]＝端末 TUI・自動化）と、各種 **[[components/node]]（Node, モバイル/デスクトップ端末）** を、**WS（WebSocket, 双方向通信トランスポート）** 経由で接続する。
- 型付き WS API（リクエスト / レスポンス / サーバープッシュイベント）を公開し、`agent` `chat` `presence` `health` `heartbeat` `cron` などのイベントを発行する。

新規デバイスは [[concepts/pairing]]（ペアリング, デバイス承認とトークン発行）を通り、すべての接続に Gateway 認証が適用される。クライアントが Gateway を見つけて経路を選ぶ仕組み（Bonjour/tailnet/SSH）は **[[concepts/discovery]]**、WS の形式的なワイヤー契約は [[sources/gateway/protocol]]、localhost/LAN/tailnet をまたぐ接続の全体像は [[sources/network]] にまとまる。詳しくは [[concepts/architecture]]（俯瞰）と解説ページ [[sources/concepts/architecture]] を参照。

Gateway は WS だけでなく **[[concepts/http-api]]**（OpenAI 互換の `/v1/chat/completions`・`/v1/responses` と単一ツール `/tools/invoke`）も同一ポートで多重化でき、既存の OpenAI クライアント（Open WebUI 等）からエージェントを叩ける。モデル側はホスト型に限らず、**[[concepts/local-models]]**（LM Studio/Ollama など自前ホスト LLM。必要時だけ起動する `localService`、API 障害時のテキスト専用 CLI フォールバックを含む）として自分のマシンで完結させることもできる。

エージェントの実行面は 3 層で捉えると見通しがよい：**[[concepts/agent]]**（1 プロセスの契約＝ワークスペース/ブートストラップ/セッション）、**[[concepts/agent-loop]]**（その 1 ターンを `agent` RPC から永続化まで回す配線）、**[[concepts/agent-runtimes]]**（ターンを実行するバックエンドの層＝pi/codex/claude-cli）。なお `agent`（プロセスの契約）と `agent-runtimes`（実行バックエンド）は名前が似て混同しやすい点に注意。

エージェントの「人格と入力」は **[[concepts/agent-workspace]]**（唯一の cwd・非公開メモリ。`SOUL.md`＝[[concepts/soul]] などの標準ファイルを置く。初回準備は [[sources/start/bootstrapping|ブートストラップ]]）に集約され、それらは **[[concepts/system-prompt]]** が実行ごとに組み立てる自前プロンプトへ注入される。モデルへ送る全体が **[[concepts/context]]**（コンテキストウィンドウ）で、その組み立て・要約を握る差し替え可能スロットが **[[concepts/context-engine]]**。モデル側の認証（サブスク OAuth/PKCE・トークン管理）は **[[concepts/oauth]]**。

会話は **[[concepts/session]]**（セッション）に整理され、送信元ごとにルーティングされて [[concepts/agent-loop]] がセッション単位で直列実行する。DM 分離はプライバシー境界になり、返信先の付け替えは **[[concepts/channel-docking]]**、ツール結果の刈り込みは **[[concepts/session-pruning]]**、セッション横断やサブエージェント生成は **[[concepts/session-tool]]**。製品自体の品質検証（実トランスポート上の QA）は **[[concepts/qa-automation]]** に切り出されている。

会話を越えて持続する知識は **[[concepts/memory]]**（ワークスペースの Markdown ファイル）に置かれ、差し替え可能なバックエンド（組み込み SQLite / QMD / Honcho）で **[[concepts/memory-search]]**（ハイブリッド検索）される。応答前の先回り想起は **[[concepts/active-memory]]**、短期シグナルの長期昇格は **[[concepts/dreaming]]**、会話から推論される短期フォローアップは **[[concepts/commitments]]**。ウィンドウが埋まると **[[concepts/compaction]]** が古い会話を要約し、その直前に重要メモを [[concepts/memory]] へ退避する。

1 つの Gateway は **[[concepts/multi-agent]]**（マルチエージェント）で複数の分離された頭脳を並立させ、バインディングで受信メッセージを振り分ける。これを組織デプロイへ拡張したのが **[[concepts/delegate-architecture]]**（代理エージェント）、並列ワークを希少リソースの設計問題として扱うのが **[[concepts/parallel-specialist-lanes]]**。接続状況の軽量ビューは **[[concepts/presence]]**（macOS の Instances タブ）。

受信から返信までの処理は **[[concepts/messages]]** パイプラインに集約される：ルーティング → セッション解決 → **[[concepts/queue]]**（レーン対応キュー。実行中ターンへの割り込みは `steer`）→ エージェント実行 → **[[concepts/streaming]]**（ブロック/プレビューの 2 層）で送信。長いターンは **[[concepts/progress-drafts]]** で 1 つの作業中下書きとして見せ、一時障害は **[[concepts/retry]]**（HTTP リクエスト単位）で吸収する。

これらすべての振る舞いは **[[concepts/configuration]]**（単一の `openclaw.json`／JSON5、厳格検証＋ホットリロード）で宣言的に設定し、[[components/gateway]] をサービスとして起動・監視する手順は運用手順書（[[sources/gateway/gateway]]）にまとまっている。**[[concepts/authentication]]** は紛らわしい 3 つの認証レイヤー（接続認証・モデル認証・デバイスペアリング）を区別し、その秘密値は **[[concepts/secrets]]**（SecretRef で env/file/exec から外部化）が支える。OpenClaw 全体の安全姿勢は **[[concepts/security]]**（「Gateway ごとに 1 つの信頼境界＝パーソナルアシスタントモデル」「知能より前にアクセス制御」）が定義し、その主要な防御が **[[concepts/sandboxing]]**（ツール実行の分離。サンドボックス／ツールポリシー／昇格の 3 制御を区別する）。

自発的な働きは **[[concepts/automation]]**（バックグラウンド作業の総称）に束ねられる：正確な時刻の **[[concepts/cron]]**、おおよそ定期の **[[concepts/heartbeat]]**、推論された **[[concepts/commitments]]**、切り離し作業を追う **[[concepts/tasks]]**（＋Task Flow）、ライフサイクルに反応する **[[concepts/hooks]]**、全セッションに注入される Standing Orders——を用途で選び分ける。エージェントが実マシンを操作する最前線が **[[concepts/exec]]**（シェル実行と承認・昇格）で、`exec` は読み取り専用にならないため [[concepts/security]] の承認設計が要になる。運用面では、**[[concepts/heartbeat]]** が自発的なチェックインや commitments/dreaming の配信を駆動し、何が起きているかは **[[concepts/logging]]**（ファイル/コンソールログ）・**[[concepts/observability]]**（Prometheus/OpenTelemetry によるメトリクス/トレース）・**[[concepts/diagnostics]]**（health・doctor・troubleshooting・診断 zip）で観測・修復する。さらに敵対的脅威を体系化した **[[concepts/threat-model]]**（MITRE ATLAS の 5 信頼境界・形式検証）が、これらの防御の優先順位を与える。

端末側では **[[components/node]]**（`role: node` のコンパニオン）が、カメラ・画面・位置・OS コマンド（`system.run`）といった能力を `node.invoke` で公開し、別マシンで `exec` を走らせる「ノードホスト」にもなる。受信した画像/音声/動画は返信前に **[[concepts/media-understanding]]**（`tools.media`）がテキスト要約し、聞く/話す体験は **[[concepts/voice]]**（Talk ループ・TTS・Voice Wake）が担う。これらはすべてテキストと同じセッション・認証・ツールポリシーの上に乗る。Gateway を別ホストに置いて外から使う運用は **[[concepts/remote-access]]**（loopback ＋ SSH トンネル / Tailscale Serve・Funnel）にまとまる。

これだけの機能は単一バイナリに詰め込まれているのではなく、**[[components/plugin-system]]**（プラグインシステム）の上に乗る。チャネル・モデルプロバイダー・エージェントハーネス（[[sources/plugins/codex-harness]]）・音声（[[sources/plugins/voice-call]] / [[sources/plugins/google-meet]]）・メモリバックエンド（[[sources/plugins/memory-lancedb]] / [[sources/plugins/memory-wiki]]）まで、多くは**コア Plugin またはバンドル**として実装され、外部 Plugin は **[[components/clawhub]]**（Plugin/Skill マーケットプレイス）から入る。エージェントの「できること」を増やす道は、ツール（呼べるアクション）・**[[concepts/skills]]**（`SKILL.md` でやり方を教える指示パック）・Plugins（ランタイム機能）の 3 つに整理される（[[sources/tools/tools]]）。代表的なツール面として、Web から情報を取る **[[concepts/web-search]]**（web_search/x_search/web_fetch ＋ 12 の差し替え可能プロバイダー）や、Claude Code 等の外部コーディングハーネスを丸ごと走らせる **[[concepts/acp]]**（Agent Client Protocol）があり、後者は [[concepts/agent-runtimes]] の「もう一つのランタイム」でもある。これらの面と人間との接点が **[[concepts/slash-commands]]**（`/...` でセッションや Gateway を決定論的に操作する明示的な制御面）であり、`.prose` でマルチエージェントを書き下す [[sources/prose]] のようなワークフロー Plugin もこの上に乗る。

## 主要カテゴリ

- **[[index|概念 (concepts)]]** — architecture / agent-loop / memory / queue / session / pairing / channel-routing / model-providers など、横断的な仕組み。
- **構成要素 (components)** — Gateway / Node / WebChat / Control UI / CLI / ClawHub など、OpenClaw を構成する具体的サブシステム。
- **チャネル (channels)** — [[concepts/channel-routing]]（＋グループ挙動の [[concepts/groups]]）の下で [[channels/slack]] / [[channels/telegram]] / [[channels/whatsapp]] / [[channels/discord]] など 25+ のメッセージング統合。
- **プロバイダー (providers)** — [[concepts/model-providers]] の下で [[providers/anthropic]] / [[providers/openai]] / [[providers/google]] / [[providers/ollama]] など 50+ のモデル提供元。

## 現状

`concepts` セクションを順次取り込み中。これまでに「Gateway を中心とした接続構成」（[[concepts/architecture]] ＋ [[components/gateway]] / [[components/node]]）、「エージェント実行の 3 層」（[[concepts/agent]] / [[concepts/agent-loop]] / [[concepts/agent-runtimes]]）、「人格・コンテキスト・認証」（[[concepts/agent-workspace]] / [[concepts/soul]] / [[concepts/system-prompt]] / [[concepts/context]] / [[concepts/context-engine]] / [[concepts/oauth]]）、「会話セッション」（[[concepts/session]] / [[concepts/session-pruning]] / [[concepts/session-tool]] / [[concepts/channel-docking]]）、「メモリと長期記憶」（[[concepts/memory]] ＋ 各バックエンド / [[concepts/memory-search]] / [[concepts/active-memory]] / [[concepts/dreaming]] / [[concepts/commitments]] / [[concepts/compaction]]）、「マルチエージェントと委任」（[[concepts/multi-agent]] / [[concepts/delegate-architecture]] / [[concepts/parallel-specialist-lanes]] / [[concepts/presence]]）、「メッセージ処理とストリーミング」（[[concepts/messages]] / [[concepts/queue]] / [[concepts/streaming]] / [[concepts/progress-drafts]] / [[concepts/retry]]）の骨格が揃い、製品の品質検証（[[concepts/qa-automation]]）と初回セットアップ（`start/`）にも着手した。これで `concepts` セクションはほぼ網羅できた。加えて **`gateway/` セクション**（設定・運用・認証・シークレット・観測/診断・セキュリティ/サンドボックス）をほぼ網羅し、[[concepts/configuration]]・[[concepts/authentication]]・[[concepts/secrets]]・[[concepts/heartbeat]]・[[concepts/logging]]・[[concepts/observability]]・[[concepts/diagnostics]]・[[concepts/security]]・[[concepts/sandboxing]] と運用手順書（[[sources/gateway/gateway]]）を取り込んだ。さらに **`gateway/` の接続・API 系**として「ネットワークと信頼」（[[concepts/pairing]] / [[concepts/discovery]] ＋ [[sources/gateway/protocol]] / [[sources/gateway/bonjour]] / [[sources/network]]）、「OpenAI 互換 HTTP サーフェス」（[[concepts/http-api]] ＝ chat completions / responses / tools-invoke）、「ローカルモデル」（[[concepts/local-models]] ＝ 登録・オンデマンド起動・CLI フォールバック）を取り込み、これで `gateway/` セクションはほぼ網羅できた。続けて **`nodes/` セクション**（[[components/node]] の大幅拡充＋ camera/audio/images/talk/voicewake/location/troubleshooting）と、横断概念の **[[concepts/media-understanding]]**（受信メディア理解）・**[[concepts/voice]]**（Talk/TTS/Voice Wake）・**[[concepts/remote-access]]**（SSH/Tailscale）、**`tools/`**（[[sources/tools/tts]] / [[sources/tools/video-generation]]）、新しい **`security/` セクション**（[[concepts/threat-model]] ＝ MITRE ATLAS / 形式検証 / [[sources/security/network-proxy]]）を取り込んだ。さらに **`web/` セクション**として人間が話す 3 つのクライアント UI——[[components/control-ui]]（ブラウザー管理ダッシュボード）・[[components/webchat]]（ネイティブチャット）・[[components/cli]]（端末 TUI）——を構成要素として起こした。続けて **`tools/`・`plugins/` セクション**を取り込み、拡張機構 **[[components/plugin-system]]** とマーケットプレイス **[[components/clawhub]]** を構成要素化、Codex ハーネス・音声通話/Google Meet・LanceDB/Memory Wiki・Webhook・oc-path などの個別 Plugin と、最初のチャネルページ **[[channels/zalouser]]** も収録した。続けて **`tools/` の Skills 群とスラッシュコマンド**（[[concepts/skills]] / [[concepts/slash-commands]]）、**Plugin SDK 群**（[[sources/plugins/building-plugins]]・[[sources/plugins/hooks]]・チャネル/プロバイダー/CLI バックエンドの構築ガイド）、ワークフロー Plugin [[sources/prose]]（OpenProse）を取り込んだ。さらに **`tools/` のツール群**（[[concepts/exec]]＝exec/承認/昇格、メディア生成・diffs・apply-patch・lobster・tool-search・trajectory・thinking 等）と **`automation/` セクション**（[[concepts/automation]] ＋ [[concepts/cron]]・[[concepts/tasks]]・[[concepts/hooks]]・Standing Orders）を取り込み、これで `tools/` と `automation/` の骨格が揃った。さらに **Web ツール群**（[[concepts/web-search]] ＝ web_search/x_search/web_fetch ＋ Brave/Perplexity/Tavily ほか 12 プロバイダー）と **マルチエージェント/ACP 系**（[[concepts/acp]]・サブエージェント・steer・エージェントごとのサンドボックス）を取り込み、`tools/` セクションはほぼ網羅できた。そして待望の **channels/・providers/・platforms/ セクション**を取り込み、2 つのハブ概念 **[[concepts/channel-routing]]**（25+ チャットチャネル）と **[[concepts/model-providers]]**（50+ モデルプロバイダー、＋[[concepts/model-failover]]）を起こした。主要チャネル（[[channels/discord]]/[[channels/slack]]/[[channels/telegram]]/[[channels/whatsapp]]）とプロバイダー（[[providers/anthropic]]/[[providers/openai]]/[[providers/google]]/[[providers/ollama]]/[[providers/bedrock]]/[[providers/litellm]]）、OS 別の [[sources/platforms/platforms]] も収録。加えて `channels/` の横断ドキュメント群（[[concepts/groups]]＝グループ挙動/メンションゲート/アクセスグループ/ブロードキャスト、決定的な [[sources/channels/channel-routing]]、DM＋Node の [[sources/channels/pairing]]、診断/位置/QA チャネル）を取り込んだ。今後は残るチャネル/プロバイダーの個別ページ・cli/・clawhub/・install/ セクションを取り込み、このページに主要な接続関係を反映していく。
