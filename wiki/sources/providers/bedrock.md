---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/bedrock
source_path: raw/docs/providers/bedrock.md
doc_section: providers
title: "Amazon Bedrock"
ingested: 2026-06-14
tags: [provider, bedrock, aws, converse, credential-chain, enterprise]
related:
  - "[[providers/bedrock]]"
  - "[[concepts/model-providers]]"
---

# Amazon Bedrock（解説）

> 原典: `raw/docs/providers/bedrock.md` ・ https://docs.openclaw.ai/ja-JP/providers/bedrock

## 一言まとめ

Amazon Bedrock のモデルを pi-ai の **Bedrock Converse** ストリーミングプロバイダー経由で使う。認証は API キーでなく **AWS SDK の既定認証情報チェーン**（プロバイダー ID `amazon-bedrock`）。

## 位置づけ

[[providers/bedrock]] の詳細ソース。[[concepts/model-providers]] のクラウド/エンタープライズ系プロバイダー。AWS 上で Claude 等を動かしたい組織向け。

## 仕組み・ふるまい

- プロバイダー `amazon-bedrock`、Bedrock Converse ストリーミング API。AWS 認証情報チェーン（環境変数・プロファイル・IAM ロール等）で認証。

## 設定・使い方の要点

- AWS 認証情報（`AWS_ACCESS_KEY_ID`/プロファイル/IAM ロール）＋リージョン。`agents.defaults.model.primary: "amazon-bedrock/<model>"`。

## 注意点・落とし穴

- ⚠️ API キーでなく **AWS 認証情報チェーン**——他プロバイダーと認証モデルが異なる。リージョンとモデルアクセス権の有効化が必要。

## 用語と略称

- **Bedrock** = AWS のマネージド LLM サービス
- **Converse** = Bedrock の統一チャット API
- **認証情報チェーン** = AWS SDK が認証情報を探す順序
- **IAM ロール** = AWS の権限付与の仕組み

## 関連ページ

- [[providers/bedrock]] — 対応するプロバイダーページ
- [[concepts/model-providers]] / [[providers/anthropic]]
