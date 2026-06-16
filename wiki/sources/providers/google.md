---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/providers/google
source_path: raw/docs/providers/google.md
doc_section: providers
title: "Google (Gemini)"
ingested: 2026-06-14
tags: [provider, google, gemini, image, media-understanding, tts, web-search]
related:
  - "[[providers/google]]"
  - "[[concepts/model-providers]]"
  - "[[concepts/media-understanding]]"
---

# Google (Gemini)（解説）

> 原典: `raw/docs/providers/google.md` ・ https://docs.openclaw.ai/ja-JP/providers/google

## 一言まとめ

Google Plugin は Google AI Studio 経由で **Gemini モデル**へのアクセスを提供し、さらに**画像生成・メディア理解・TTS・Gemini Grounding による Web 検索**まで担う、多機能なプロバイダー。

## 位置づけ

[[providers/google]] の詳細ソース。[[concepts/model-providers]] の主要プロバイダーで、[[concepts/media-understanding]]（画像/音声/動画理解）・[[concepts/voice]]（TTS）・[[concepts/web-search]]（[[sources/tools/gemini-search]]）にも横断的に関わる。

## 仕組み・ふるまい

- Provider `google`、認証 `GEMINI_API_KEY`/`GOOGLE_API_KEY`、API は Google Gemini API。チャットに加え、画像生成（`image_generate`）・メディア理解・TTS・Web 検索を 1 プロバイダーで。
- `google/gemini-3-pro-image-preview` 等で画像生成、Gemini CLI バックエンドもある。

## 設定・使い方の要点

- `agents.defaults.model.primary: "google/..."`＋`GEMINI_API_KEY`。Web 検索は `plugins.entries.google.config.webSearch`、低優先で `models.providers.google.apiKey` を再利用。Vertex は `anthropic-vertex` や Gemini CLI 経由（[[sources/concepts/model-providers]]）。

## 用語と略称

- **Gemini** = Google のモデルファミリー
- **AI Studio** = Google の API/モデルポータル
- **Grounding** = 検索結果に基づく回答（[[concepts/web-search]]）
- **Vertex** = Google Cloud の AI プラットフォーム

## 関連ページ

- [[providers/google]] — 対応するプロバイダーページ
- [[concepts/model-providers]] / [[concepts/media-understanding]] / [[concepts/voice]] / [[sources/tools/gemini-search]]
