---
title: "Skills の作成"
source: "https://docs.openclaw.ai/ja-JP/tools/creating-skills"
author:
published:
created: 2026-06-14
description: "SKILL.md でカスタムワークスペース Skills をビルドしてテストする"
tags:
  - "clippings"
---
Skills は、エージェントがツールを使用する方法とタイミングを教えます。各 skill は、YAML frontmatter と markdown の手順を含む `SKILL.md` ファイルを持つディレクトリです。

skills の読み込み方法と優先順位については、 [Skills](https://docs.openclaw.ai/ja-JP/tools/skills) を参照してください。

## 最初の skill を作成する

- ### skill ディレクトリを作成する
	Skills はワークスペース内にあります。新しいフォルダーを作成します。
	bash
	```bash
	mkdir -p ~/.openclaw/workspace/skills/hello-world
	```
- ### SKILL.md を書く
	そのディレクトリ内に `SKILL.md` を作成します。frontmatter はメタデータを定義し、markdown 本文にはエージェント向けの手順を記載します。
	markdown
	```markdown
	---
	name: hello-world
	description: A simple skill that says hello.
	---
	 
	# Hello World Skill
	 
	When the user asks for a greeting, use the \`echo\` tool to say
	"Hello from your custom skill!".
	```
	skill の `name` には、小文字、数字、ハイフンを使ったハイフンケースを使用します。フォルダー名と frontmatter の `name` を一致させてください。
- ### ツールを追加する（任意）
	frontmatter でカスタムツールスキーマを定義することも、既存のシステムツール（ `exec` や `browser` など）を使用するようエージェントに指示することもできます。Skills は、それらが文書化するツールと一緒に plugins 内に同梱することもできます。
- ### skill を読み込む
	OpenClaw が skill を検出できるよう、新しいセッションを開始します。
	bash
	```bash
	# From chat
	/new
	 
	# Or restart the gateway
	openclaw gateway restart
	```
	skill が読み込まれたことを確認します。
	bash
	```bash
	openclaw skills list
	```
- ### テストする
	skill をトリガーするはずのメッセージを送信します。
	bash
	```bash
	openclaw agent --message "give me a greeting"
	```
	または、エージェントと通常どおりチャットして、挨拶を依頼します。

## skill メタデータのリファレンス

YAML frontmatter は以下のフィールドをサポートします。

| フィールド | 必須 | 説明 |
| --- | --- | --- |
| `name` | はい | 小文字、数字、ハイフンを使用する一意の識別子 |
| `description` | はい | エージェントに表示される 1 行の説明 |
| `metadata.openclaw.os` | いいえ | OS フィルター（ `["darwin"]` 、 `["linux"]` など） |
| `metadata.openclaw.requires.bins` | いいえ | PATH 上に必要なバイナリ |
| `metadata.openclaw.requires.config` | いいえ | 必要な設定キー |

## ベストプラクティス

- **簡潔にする** — AI としてどう振る舞うかではなく、 *何を* するかをモデルに指示します
- **安全性を優先する** — skill が `exec` を使用する場合、信頼できない入力から任意のコマンドインジェクションを許可しないプロンプトにしてください
- **ローカルでテストする** — 共有する前に `openclaw agent --message "..."` を使ってテストします
- **ClawHub を使う** — [ClawHub](https://clawhub.ai/) で skills を閲覧し、貢献します

## skills の場所

| 場所 | 優先順位 | スコープ |
| --- | --- | --- |
| `\<workspace\>/skills/` | 最高 | エージェントごと |
| `\<workspace\>/.agents/skills/` | 高 | ワークスペースのエージェントごと |
| `~/.agents/skills/` | 中 | 共有エージェントプロファイル |
| `~/.openclaw/skills/` | 中 | 共有（すべてのエージェント） |
| OpenClaw に同梱 | 低 | グローバル |
| `skills.load.extraDirs` | 最低 | カスタム共有フォルダー |

## 関連

- [Skills リファレンス](https://docs.openclaw.ai/ja-JP/tools/skills) — 読み込み、優先順位、ゲートルール
- [Skills 設定](https://docs.openclaw.ai/ja-JP/tools/skills-config) — `skills.*` 設定スキーマ
- [ClawHub](https://docs.openclaw.ai/ja-JP/clawhub) — 公開 skill レジストリ
- [Plugins の構築](https://docs.openclaw.ai/ja-JP/plugins/building-plugins) — plugins は skills を同梱できます