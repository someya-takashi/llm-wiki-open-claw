---
title: "差分"
source: "https://docs.openclaw.ai/ja-JP/tools/diffs"
author:
published:
created: 2026-06-14
description: "エージェント向けの読み取り専用差分ビューアーおよびファイルレンダラー（任意の Plugin ツール）"
tags:
  - "clippings"
---
`diffs` は、短い組み込みシステムガイダンスと付随するSkillを備えた任意のPluginツールで、変更内容をエージェント向けの読み取り専用diff成果物に変換します。

次のいずれかを受け付けます。

次を返せます。

- キャンバス表示用のGatewayビューアURL
- メッセージ配信用のレンダリング済みファイルパス（PNGまたはPDF）
- 1回の呼び出しで両方の出力

有効にすると、このPluginは簡潔な使用ガイダンスをシステムプロンプト領域に追加し、エージェントがより詳しい手順を必要とする場合のために詳細なSkillも公開します。

## クイックスタート

- ### プラグインをインストール
	bash
	```bash
	openclaw plugins install diffs
	```
- ### プラグインを有効化
	json5
	```
	{
	  plugins: {
	    entries: {
	      diffs: {
	        enabled: true,
	      },
	    },
	  },
	}
	```
- ### モードを選択
	### view
	キャンバス優先のフロー: エージェントは `mode: "view"` で `diffs` を呼び出し、 `canvas present` で `details.viewerUrl` を開きます。
	### file
	チャットファイル配信: エージェントは `mode: "file"` で `diffs` を呼び出し、 `path` または `filePath` を使って `message` で `details.filePath` を送信します。
	### both
	組み合わせ: エージェントは `mode: "both"` で `diffs` を呼び出し、1回の呼び出しで両方の成果物を取得します。

## 組み込みシステムガイダンスを無効化する

`diffs` ツールを有効にしたまま、組み込みのシステムプロンプトガイダンスを無効にしたい場合は、 `plugins.entries.diffs.hooks.allowPromptInjection` を `false` に設定します。

json5

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        hooks: {
          allowPromptInjection: false,
        },
      },
    },
  },
}
```

これにより、Plugin、ツール、付随するSkillは利用可能なまま、diffs Pluginの `before_prompt_build` フックがブロックされます。

ガイダンスとツールの両方を無効にしたい場合は、代わりにPluginを無効にします。

## 典型的なエージェントワークフロー

- ### diffsを呼び出す
	エージェントが入力を指定して `diffs` ツールを呼び出します。
- ### detailsを読む
	エージェントがレスポンスから `details` フィールドを読みます。
- ### 提示する
	エージェントは `canvas present` で `details.viewerUrl` を開くか、 `path` または `filePath` を使って `message` で `details.filePath` を送信するか、またはその両方を行います。

## 入力例

### 変更前と変更後

json

```json
{
  "before": "# Hello\n\nOne",
  "after": "# Hello\n\nTwo",
  "path": "docs/example.md",
  "mode": "view"
}
```

### パッチ

json

```json
{
  "patch": "diff --git a/src/example.ts b/src/example.ts\n--- a/src/example.ts\n+++ b/src/example.ts\n@@ -1 +1 @@\n-const x = 1;\n+const x = 2;\n",
  "mode": "both"
}
```

## ツール入力リファレンス

明記されていない限り、すべてのフィールドは任意です。

元のテキスト。 `patch` が省略されている場合、 `after` とともに必須です。

更新後のテキスト。 `patch` が省略されている場合、 `before` とともに必須です。

統一diffテキスト。 `before` および `after` と相互排他的です。

変更前後モードで表示するファイル名。

変更前後モード用の言語上書きヒント。不明な値はプレーンテキストにフォールバックします。

ビューアタイトルの上書き。

出力モード。Pluginのデフォルト `defaults.mode` が既定です。非推奨エイリアス: `"image"` は `"file"` と同様に動作し、後方互換性のため現在も受け付けられます。

ビューアテーマ。Pluginのデフォルト `defaults.theme` が既定です。

diffレイアウト。Pluginのデフォルト `defaults.layout` が既定です。

完全なコンテキストが利用可能な場合に、未変更セクションを展開します。呼び出しごとのオプションのみです（Pluginのデフォルトキーではありません）。

レンダリング済みファイル形式。Pluginのデフォルト `defaults.fileFormat` が既定です。

PNGまたはPDFレンダリング用の品質プリセット。

デバイススケールの上書き（ `1` - `4` ）。

CSSピクセル単位の最大レンダリング幅（ `640` - `2400` ）。

ビューアとスタンドアロンファイル出力の成果物TTL（秒）。最大21600。

ビューアURLのオリジン上書き。Pluginの `viewerBaseUrl` を上書きします。 `http` または `https` である必要があり、クエリ/ハッシュは使用できません。

レガシー入力エイリアス

後方互換性のため現在も受け付けられます。

- `format` -> `fileFormat`
- `imageFormat` -> `fileFormat`
- `imageQuality` -> `fileQuality`
- `imageScale` -> `fileScale`
- `imageMaxWidth` -> `fileMaxWidth`
検証と制限
- `before` と `after` はそれぞれ最大512 KiB。
- `patch` は最大2 MiB。
- `path` は最大2048バイト。
- `lang` は最大128バイト。
- `title` は最大1024バイト。
- パッチ複雑性の上限: 最大128ファイル、合計120000行。
- `patch` と `before` または `after` の併用は拒否されます。
- レンダリング済みファイルの安全上限（PNGとPDFに適用）:
	- `fileQuality: "standard"`: 最大8 MP（8,000,000レンダリングピクセル）。
		- `fileQuality: "hq"`: 最大14 MP（14,000,000レンダリングピクセル）。
		- `fileQuality: "print"`: 最大24 MP（24,000,000レンダリングピクセル）。
		- PDFにはさらに最大50ページの上限があります。

## 出力details契約

ツールは `details` 配下に構造化メタデータを返します。

ビューアフィールド

ビューアを作成するモードで共有されるフィールド:

- `artifactId`
- `viewerUrl`
- `viewerPath`
- `title`
- `expiresAt`
- `inputKind`
- `fileCount`
- `mode`
- `context` （利用可能な場合は `agentId` 、 `sessionId` 、 `messageChannel` 、 `agentAccountId` ）
ファイルフィールド

PNGまたはPDFがレンダリングされた場合のファイルフィールド:

- `artifactId`
- `expiresAt`
- `filePath`
- `path` （メッセージツール互換性のため、 `filePath` と同じ値）
- `fileBytes`
- `fileFormat`
- `fileQuality`
- `fileScale`
- `fileMaxWidth`
互換性エイリアス

既存の呼び出し元向けに次も返されます。

- `format` （ `fileFormat` と同じ値）
- `imagePath` （ `filePath` と同じ値）
- `imageBytes` （ `fileBytes` と同じ値）
- `imageQuality` （ `fileQuality` と同じ値）
- `imageScale` （ `fileScale` と同じ値）
- `imageMaxWidth` （ `fileMaxWidth` と同じ値）

モード動作の概要:

| モード | 返されるもの |
| --- | --- |
| `"view"` | ビューアフィールドのみ。 |
| `"file"` | ファイルフィールドのみ。ビューア成果物はありません。 |
| `"both"` | ビューアフィールドとファイルフィールド。ファイルレンダリングに失敗した場合でも、ビューアは `fileError` と `imageError` エイリアス付きで返されます。 |

## 折りたたまれた未変更セクション

- ビューアには `N unmodified lines` のような行を表示できます。
- それらの行の展開コントロールは条件付きであり、すべての入力種別で保証されるわけではありません。
- 展開コントロールは、レンダリングされたdiffに展開可能なコンテキストデータがある場合に表示されます。これは変更前後入力で一般的です。
- 多くの統一パッチ入力では、省略されたコンテキスト本文がパース済みパッチハンクで利用できないため、展開コントロールなしで行が表示される場合があります。これは想定された動作です。
- `expandUnchanged` は、展開可能なコンテキストが存在する場合にのみ適用されます。

## Pluginデフォルト

Plugin全体のデフォルトを `~/.openclaw/openclaw.json` に設定します。

json5

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          defaults: {
            fontFamily: "Fira Code",
            fontSize: 15,
            lineSpacing: 1.6,
            layout: "unified",
            showLineNumbers: true,
            diffIndicators: "bars",
            wordWrap: true,
            background: true,
            theme: "dark",
            fileFormat: "png",
            fileQuality: "standard",
            fileScale: 2,
            fileMaxWidth: 960,
            mode: "both",
            ttlSeconds: 21600,
          },
        },
      },
    },
  },
}
```

サポートされるデフォルト:

- `fontFamily`
- `fontSize`
- `lineSpacing`
- `layout`
- `showLineNumbers`
- `diffIndicators`
- `wordWrap`
- `background`
- `theme`
- `fileFormat`
- `fileQuality`
- `fileScale`
- `fileMaxWidth`
- `mode`
- `ttlSeconds`

明示的なツールパラメーターはこれらのデフォルトを上書きします。

### 永続的なビューアURL設定

ツール呼び出しで `baseUrl` が渡されない場合に返されるビューアリンク用の、Plugin所有のフォールバック。 `http` または `https` である必要があり、クエリ/ハッシュは使用できません。

json5

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          viewerBaseUrl: "https://gateway.example.com/openclaw",
        },
      },
    },
  },
}
```

## セキュリティ設定

`false`: ビューアルートへの非ループバックリクエストは拒否されます。 `true`: トークン付きパスが有効な場合、リモートビューアが許可されます。

json5

```
{
  plugins: {
    entries: {
      diffs: {
        enabled: true,
        config: {
          security: {
            allowRemoteViewer: false,
          },
        },
      },
    },
  },
}
```

## 成果物のライフサイクルとストレージ

- 成果物は一時サブフォルダー `$TMPDIR/openclaw-diffs` 配下に保存されます。
- ビューア成果物メタデータには次が含まれます。
	- ランダムな成果物ID（20桁の16進文字）
		- ランダムなトークン（48桁の16進文字）
		- `createdAt` と `expiresAt`
		- 保存された `viewer.html` パス
- 指定されていない場合、デフォルトの成果物TTLは30分です。
- 受け付けられる最大ビューアTTLは6時間です。
- クリーンアップは成果物作成後に機会的に実行されます。
- 期限切れの成果物は削除されます。
- メタデータがない場合、フォールバッククリーンアップにより24時間より古い古いフォルダーが削除されます。

## ビューアURLとネットワーク動作

ビューアルート:

- `/plugins/diffs/view/{artifactId}/{token}`

ビューアアセット:

- `/plugins/diffs/assets/viewer.js`
- `/plugins/diffs/assets/viewer-runtime.js`

ビューアドキュメントはこれらのアセットをビューアURLからの相対パスとして解決するため、任意の `baseUrl` パスプレフィックスは両方のアセットリクエストにも保持されます。

URL構築の動作:

- ツール呼び出しの `baseUrl` が指定されている場合、厳密な検証後に使用されます。
- それ以外でPluginの `viewerBaseUrl` が設定されている場合は、それが使用されます。
- どちらの上書きもない場合、ビューアURLはlocal loopback `127.0.0.1` が既定です。
- Gatewayバインドモードが `custom` で `gateway.customBindHost` が設定されている場合、そのホストが使用されます。

`baseUrl` のルール:

- `http://` または `https://` である必要があります。
- クエリとハッシュは拒否されます。
- オリジンに任意のベースパスを加えたものが許可されます。

## セキュリティモデル

ビューアの堅牢化
- デフォルトではループバック専用。
- 厳格な ID とトークン検証を伴う、トークン化されたビューアパス。
- ビューアレスポンスの CSP:
	- `default-src 'none'`
		- スクリプトとアセットは self からのみ
		- 外向きの `connect-src` なし
- リモートアクセスが有効な場合のリモートミスのスロットリング:
	- 60 秒あたり 40 回の失敗
		- 60 秒のロックアウト（ `429 Too Many Requests` ）
ファイルレンダリングの堅牢化
- スクリーンショット用ブラウザリクエストのルーティングはデフォルト拒否。
- `http://127.0.0.1/plugins/diffs/assets/*` のローカルビューアアセットのみ許可されます。
- 外部ネットワークリクエストはブロックされます。

## ファイルモードのブラウザ要件

`mode: "file"` と `mode: "both"` には Chromium 互換ブラウザが必要です。

解決順序:

- ### 設定
	OpenClaw 設定の `browser.executablePath` 。
- ### 環境変数
	- `OPENCLAW_BROWSER_EXECUTABLE_PATH`
	- `BROWSER_EXECUTABLE_PATH`
	- `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH`
- ### プラットフォームフォールバック
	プラットフォームのコマンド/パス検出フォールバック。

よくある失敗メッセージ:

- `Diff PNG/PDF rendering requires a Chromium-compatible browser...`

Chrome、Chromium、Edge、または Brave をインストールするか、上記の実行可能パスオプションのいずれかを設定して修正します。

## トラブルシューティング

入力検証エラー
- `Provide patch or both before and after text.` — `before` と `after` の両方を含めるか、 `patch` を指定します。
- `Provide either patch or before/after input, not both.` — 入力モードを混在させないでください。
- `Invalid baseUrl: ...` — 任意のパスを含む `http(s)` オリジンを使用し、クエリ/ハッシュは含めないでください。
- `{field} exceeds maximum size (...)` — ペイロードサイズを減らします。
- 大きなパッチの拒否 — パッチファイル数または総行数を減らします。
ビューアのアクセシビリティ
- ビューア URL はデフォルトで `127.0.0.1` に解決されます。
- リモートアクセスのシナリオでは、次のいずれかを行います:
	- Plugin の `viewerBaseUrl` を設定する、または
		- ツール呼び出しごとに `baseUrl` を渡す、または
		- `gateway.bind=custom` と `gateway.customBindHost` を使用する
- `gateway.trustedProxies` に同一ホストのプロキシ（例: Tailscale Serve）用のループバックが含まれている場合、転送されたクライアント IP ヘッダーのない生のループバックビューアリクエストは、設計上 fail closed になります。
- そのプロキシトポロジでは:
	- 添付ファイルだけが必要な場合は `mode: "file"` または `mode: "both"` を優先する、または
		- 共有可能なビューア URL が必要な場合は、意図的に `security.allowRemoteViewer` を有効にし、Plugin の `viewerBaseUrl` を設定するか、プロキシ/公開 `baseUrl` を渡す
- 外部ビューアアクセスを意図する場合のみ、 `security.allowRemoteViewer` を有効にします。
変更されていない行に展開ボタンがない

パッチが展開可能なコンテキストを持っていない場合、パッチ入力でこれが発生することがあります。これは想定どおりであり、ビューアの失敗を示すものではありません。

Artifact が見つかりません
- TTL により Artifact が期限切れになりました。
- トークンまたはパスが変更されました。
- クリーンアップにより古いデータが削除されました。

## 運用ガイダンス

- キャンバスでのローカル対話型レビューには `mode: "view"` を優先します。
- 添付ファイルが必要な外向きチャットチャネルには `mode: "file"` を優先します。
- デプロイでリモートビューア URL が必要な場合を除き、 `allowRemoteViewer` は無効のままにします。
- 機密性の高い diff には、明示的に短い `ttlSeconds` を設定します。
- 必須でない場合は、diff 入力にシークレットを送信しないでください。
- チャネルが画像を強く圧縮する場合（例: Telegram や WhatsApp）は、PDF 出力（ `fileFormat: "pdf"` ）を優先します。

> [!note] Note
> **Note**
> 
> Diff レンダリングエンジンは [Diffs](https://diffs.com/) により提供されています。

## 関連

- [ブラウザ](https://docs.openclaw.ai/ja-JP/tools/browser)
- [Plugin](https://docs.openclaw.ai/ja-JP/tools/plugin)