---
title: "概要"
source: "https://docs.openclaw.ai/ja-JP/tools"
author:
published:
created: 2026-06-14
description: "OpenClaw のツール、Skills、Plugin の概要: エージェントが呼び出せるものと、それらを拡張する方法"
tags:
  - "clippings"
---
このページは、適切な Capabilities サーフェスを選ぶために使用します。 **ツール** は呼び出し可能なアクションであり、 **Skills** はエージェントに作業方法を教え、 **plugins** はツール、プロバイダー、チャネル、フック、パッケージ化された Skills などのランタイム機能を追加します。

これは概要とルーティングのページです。網羅的なツールポリシー、デフォルト、グループメンバーシップ、プロバイダー制限、設定フィールドについては、 [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools) を使用してください。

## ここから始める

ほとんどのエージェントでは、まず組み込みツールカテゴリから始め、その後、エージェントに見せるツールを減らす必要がある場合や、明示的なホストアクセスが必要な場合にだけポリシーを調整します。

| 必要なこと | 最初に使うもの | 次に読むもの |
| --- | --- | --- |
| 既存の機能でエージェントに動作させる | [組み込みツール](#built-in-tool-categories) | [ツールカテゴリ](#built-in-tool-categories) |
| エージェントが呼び出せるものを制御する | [ツールポリシー](#configure-access-and-approvals) | [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools) |
| エージェントにワークフローを教える | [Skills](#choose-tools-skills-or-plugins) | [Skills](https://docs.openclaw.ai/ja-JP/tools/skills) と [Skills の作成](https://docs.openclaw.ai/ja-JP/tools/creating-skills) |
| 新しいインテグレーションまたはランタイムサーフェスを追加する | [Plugins](#extend-capabilities) | [Plugins](https://docs.openclaw.ai/ja-JP/tools/plugin) と [plugins のビルド](https://docs.openclaw.ai/ja-JP/plugins/building-plugins) |
| 後で、またはバックグラウンドで作業を実行する | [Automation](https://docs.openclaw.ai/ja-JP/automation) | [Automation の概要](https://docs.openclaw.ai/ja-JP/automation) |
| 複数のエージェントまたはハーネスを調整する | [サブエージェント](https://docs.openclaw.ai/ja-JP/tools/subagents) | [ACP エージェント](https://docs.openclaw.ai/ja-JP/tools/acp-agents) と [エージェント送信](https://docs.openclaw.ai/ja-JP/tools/agent-send) |
| 大規模な PI ツールカタログを検索する | [Tool Search](https://docs.openclaw.ai/ja-JP/tools/tool-search) | [Tool Search](https://docs.openclaw.ai/ja-JP/tools/tool-search) |

## ツール、Skills、plugins を選ぶ

- ### エージェントが行動する必要がある場合はツールを使う
	ツールは、エージェントが呼び出せる型付き関数です。たとえば `exec` 、 `browser` 、 `web_search` 、 `message` 、 `image_generate` などです。エージェントがデータを読み取る、 ファイルを変更する、メッセージを送信する、プロバイダーを呼び出す、または別のシステムを操作する必要がある場合にツールを使います。 表示されるツールは、構造化された関数定義としてモデルに送信されます。
	モデルが見るのは、アクティブなプロファイル、許可/拒否ポリシー、 プロバイダー制限、サンドボックス状態、チャネル権限、 plugin の可用性を通過したツールだけです。
- ### エージェントに指示が必要な場合は Skill を使う
	Skill は、エージェントプロンプトに読み込まれる `SKILL.md` 指示パックです。 エージェントが必要なツールをすでに持っているが、再現可能なワークフロー、 レビュー基準、コマンドシーケンス、または操作上の制約が必要な場合に Skill を使います。
	Skills は、ワークスペース、共有 Skill ディレクトリ、管理対象の OpenClaw Skill ルート、または plugin パッケージに置くことができます。
	[Skills](https://docs.openclaw.ai/ja-JP/tools/skills) | [Skills の作成](https://docs.openclaw.ai/ja-JP/tools/creating-skills) | [Skills 設定](https://docs.openclaw.ai/ja-JP/tools/skills-config)
- ### OpenClaw に新しい機能が必要な場合は plugin を使う
	plugin は、ツール、Skills、チャネル、モデルプロバイダー、音声、リアルタイム音声、 メディア生成、Web 検索、Web 取得、フック、その他のランタイム機能を追加できます。 機能にコード、資格情報、ライフサイクルフック、マニフェストメタデータ、 またはインストール可能なパッケージングがある場合に plugin を使います。既存の plugins は、ClawHub、npm、git、ローカルディレクトリ、または アーカイブからインストールできます。
	[plugins のインストールと設定](https://docs.openclaw.ai/ja-JP/tools/plugin) | [plugins のビルド](https://docs.openclaw.ai/ja-JP/plugins/building-plugins) | [Plugin SDK](https://docs.openclaw.ai/ja-JP/plugins/sdk-overview)

## 組み込みツールカテゴリ

この表には、サーフェスを認識できるよう代表的なツールを示しています。これは 完全なポリシーリファレンスではありません。正確なグループ、デフォルト、許可/拒否 セマンティクスについては、 [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools) を使用してください。

| カテゴリ | エージェントに必要なこと | 代表的なツール | 次に読むもの |
| --- | --- | --- | --- |
| ランタイム | コマンドを実行する、プロセスを管理する、またはプロバイダー支援の Python 解析を使う | `exec`, `process`, `code_execution` | [Exec](https://docs.openclaw.ai/ja-JP/tools/exec), [コード実行](https://docs.openclaw.ai/ja-JP/tools/code-execution) |
| ファイル | ワークスペースファイルを読み取り、変更する | `read`, `write`, `edit`, `apply_patch` | [Apply patch](https://docs.openclaw.ai/ja-JP/tools/apply-patch) |
| Web | Web を検索する、X 投稿を検索する、または読み取り可能なページ内容を取得する | `web_search`, `x_search`, `web_fetch` | [Web ツール](https://docs.openclaw.ai/ja-JP/tools/web), [Web fetch](https://docs.openclaw.ai/ja-JP/tools/web-fetch) |
| ブラウザ | ブラウザセッションを操作する | `browser` | [ブラウザ](https://docs.openclaw.ai/ja-JP/tools/browser) |
| メッセージングとチャネル | 返信またはチャネルアクションを送信する | `message` | [エージェント送信](https://docs.openclaw.ai/ja-JP/tools/agent-send) |
| セッションとエージェント | セッションを調べる、作業を委任する、別の実行を誘導する、またはステータスを報告する | `sessions_*`, `subagents`, `agents_list`, `session_status` | [サブエージェント](https://docs.openclaw.ai/ja-JP/tools/subagents), [セッションツール](https://docs.openclaw.ai/ja-JP/concepts/session-tool) |
| Automation | 作業をスケジュールする、またはバックグラウンドイベントに応答する | `cron`, `heartbeat_respond` | [Automation](https://docs.openclaw.ai/ja-JP/automation) |
| Gateway とノード | Gateway 状態またはペアリングされたターゲットデバイスを調べる | `gateway`, `nodes` | [Gateway 設定](https://docs.openclaw.ai/ja-JP/gateway/configuration), [ノード](https://docs.openclaw.ai/ja-JP/nodes) |
| メディア | メディアを分析、生成、または読み上げる | `image`, `image_generate`, `music_generate`, `video_generate`, `tts` | [メディア概要](https://docs.openclaw.ai/ja-JP/tools/media-overview) |
| 大規模な PI カタログ | すべてのスキーマをモデルに送らずに、多数の対象ツールを検索して呼び出す | `tool_search_code`, `tool_search`, `tool_describe` | [Tool Search](https://docs.openclaw.ai/ja-JP/tools/tool-search) |

> [!note] Note
> **Note**
> 
> Tool Search は実験的な PI エージェントサーフェスです。Codex ハーネスの実行では、 `tools.toolSearch` の代わりに、Codex ネイティブのコードモード、ネイティブツール検索、 遅延動的ツール、ネストされたツール呼び出しを使用します。

## plugin 提供のツール

plugins は追加のツールを登録できます。plugin 作者は `api.registerTool(...)` とマニフェストの `contracts.tools` を通じてツールを接続します。契約の詳細については [Plugin SDK](https://docs.openclaw.ai/ja-JP/plugins/sdk-overview) と [Plugin マニフェスト](https://docs.openclaw.ai/ja-JP/plugins/manifest) を使用してください。

一般的な plugin 提供ツールには次があります。

- ファイルと Markdown の差分をレンダリングするための [Diffs](https://docs.openclaw.ai/ja-JP/tools/diffs)
- JSON のみのワークフローステップ用の [LLM Task](https://docs.openclaw.ai/ja-JP/tools/llm-task)
- 再開可能な承認を伴う型付きワークフロー用の [Lobster](https://docs.openclaw.ai/ja-JP/tools/lobster)
- ノイズの多い `exec` と `bash` ツール出力を圧縮するための [Tokenjuice](https://docs.openclaw.ai/ja-JP/tools/tokenjuice)
- すべてのスキーマをプロンプトに入れずに大規模なツールカタログを発見して呼び出すための [Tool Search](https://docs.openclaw.ai/ja-JP/tools/tool-search)
- ノード Canvas 制御と A2UI レンダリング用の [Canvas](https://docs.openclaw.ai/ja-JP/plugins/reference/canvas)

## アクセスと承認を設定する

ツールポリシーはモデル呼び出しの前に適用されます。ポリシーがツールを削除した場合、 モデルはそのターンでそのツールのスキーマを受け取りません。実行では、 グローバル設定、エージェントごとの設定、チャネルポリシー、プロバイダー 制限、サンドボックスルール、オーナー限定ゲート、または plugin の可用性によってツールを失うことがあります。

- [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools) では、ツールプロファイル、 許可/拒否リスト、プロバイダー固有の制限、ループ検出、 プロバイダー支援ツール設定を説明しています。
- [Exec 承認](https://docs.openclaw.ai/ja-JP/tools/exec-approvals) では、ホストコマンド承認 ポリシーを説明しています。
- [昇格 exec](https://docs.openclaw.ai/ja-JP/tools/elevated) では、サンドボックス外での制御された実行を 説明しています。
- [サンドボックス vs ツールポリシー vs 昇格](https://docs.openclaw.ai/ja-JP/gateway/sandbox-vs-tool-policy-vs-elevated) では、どのレイヤーがファイルとプロセスのアクセスを制御するかを説明しています。
- [エージェントごとのサンドボックスとツール制限](https://docs.openclaw.ai/ja-JP/tools/multi-agent-sandbox-tools) では、委任された実行に対するエージェント固有の制限を説明しています。

## 機能を拡張する

OpenClaw に実行させる必要がある作業に応じて、拡張パスを選びます。

- [Plugins](https://docs.openclaw.ai/ja-JP/tools/plugin) で既存の plugin をインストールまたは管理します。
- [plugins のビルド](https://docs.openclaw.ai/ja-JP/plugins/building-plugins) で新しいインテグレーション、プロバイダー、チャネル、ツール、またはフックを構築します。
- [Skills](https://docs.openclaw.ai/ja-JP/tools/skills) と [Skills の作成](https://docs.openclaw.ai/ja-JP/tools/creating-skills) で再利用可能なエージェント指示を追加または調整します。
- ワークフローが plugin 配布の Skill バンドルに属する場合は、 [Skill workshop](https://docs.openclaw.ai/ja-JP/plugins/skill-workshop) で再利用可能なワークフロー素材をパッケージ化します。
- 実装契約が必要な場合は、 [Plugin SDK](https://docs.openclaw.ai/ja-JP/plugins/sdk-overview) と [Plugin マニフェスト](https://docs.openclaw.ai/ja-JP/plugins/manifest) を使用します。

## 見つからないツールのトラブルシューティング

モデルがツールを見たり呼び出したりできない場合は、現在のターンの有効なポリシーから始めます。

1. [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools) で、アクティブなプロファイル、 `tools.allow` 、 `tools.deny` を確認します。
2. [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools) でプロバイダー固有の制限を確認し、選択した [モデルプロバイダー](https://docs.openclaw.ai/ja-JP/concepts/model-providers) がそのツール形状をサポートしていることを確認します。
3. [サンドボックス vs ツールポリシー vs 昇格](https://docs.openclaw.ai/ja-JP/gateway/sandbox-vs-tool-policy-vs-elevated) と [昇格 exec](https://docs.openclaw.ai/ja-JP/tools/elevated) で、チャネル権限、サンドボックス状態、昇格アクセスを確認します。
4. 所有元の plugin が [Plugins](https://docs.openclaw.ai/ja-JP/tools/plugin) でインストールされ、有効化されているかを確認します。
5. 委任された実行では、 [エージェントごとのサンドボックスとツール制限](https://docs.openclaw.ai/ja-JP/tools/multi-agent-sandbox-tools) でエージェントごとの制限を確認します。
6. 大規模な PI カタログでは、その実行が直接のツール公開を使っているのか、 [Tool Search](https://docs.openclaw.ai/ja-JP/tools/tool-search) を使っているのかを確認します。

## 関連

- cron、タスク、Heartbeat、コミットメント、フック、常設指示、Task Flow については [Automation](https://docs.openclaw.ai/ja-JP/automation)
- エージェントモデル、セッション、メモリ、マルチエージェント調整については [エージェント](https://docs.openclaw.ai/ja-JP/concepts/agent)
- 正規のツールポリシーリファレンスについては [ツールとカスタムプロバイダー](https://docs.openclaw.ai/ja-JP/gateway/config-tools)
- plugin のインストールと管理については [Plugins](https://docs.openclaw.ai/ja-JP/tools/plugin)
- plugin 作者向けリファレンスについては [Plugin SDK](https://docs.openclaw.ai/ja-JP/plugins/sdk-overview)
- Skill の読み込み順、ゲート、設定については [Skills](https://docs.openclaw.ai/ja-JP/tools/skills)
- コンパクトな PI ツールカタログ探索については [Tool Search](https://docs.openclaw.ai/ja-JP/tools/tool-search)