---
type: provider
aliases: [Amazon Bedrock, Bedrock, AWS]
tags: [model-provider, llm, bedrock, aws, enterprise]
related:
  - "[[concepts/model-providers]]"
  - "[[components/gateway]]"
sources:
  - "[[sources/providers/bedrock]]"
updated: 2026-06-14
---

# Amazon Bedrock

**Amazon Bedrock** は、AWS のマネージド LLM サービス（プロバイダー `amazon-bedrock`）。pi-ai の **Bedrock Converse** ストリーミング経由でモデルを使い、認証は API キーでなく **AWS SDK の既定認証情報チェーン**（環境変数/プロファイル/IAM ロール）。[[concepts/model-providers]] のクラウド/エンタープライズ系で、AWS 上で Claude 等を動かす組織向け。

詳細・認証は [[sources/providers/bedrock]]。

## 関連

- [[concepts/model-providers]] / [[providers/anthropic]] / [[components/gateway]]
