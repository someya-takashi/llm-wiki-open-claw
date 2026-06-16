---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/platforms/android
source_path: raw/docs/platforms/android.md
doc_section: platforms
title: "Android アプリ"
ingested: 2026-06-14
tags: [platform, android, mobile-node, companion-app, personal-data]
related:
  - "[[components/node]]"
  - "[[sources/platforms/platforms]]"
  - "[[concepts/pairing]]"
---

# Android アプリ（解説）

> 原典: `raw/docs/platforms/android.md` ・ https://docs.openclaw.ai/ja-JP/platforms/android

## 一言まとめ

Android アプリは OpenClaw の**モバイルノード**で、端末機能（カメラ・画面・位置・SMS・連絡先・カレンダー等の個人データコマンド）を Gateway 経由で公開する。一般公開前で、ソースから自前ビルド可（`apps/android`、Gradle）。

## 位置づけ

[[components/node]] の Android 実装。[[sources/platforms/platforms]] の OS 別ページで、接続は [[concepts/pairing]]。Android 固有の個人データコマンドファミリーを持つ。

## 仕組み・ふるまい

- Node として接続し、`camera.*`/`screen.*`/`location.*` に加え `sms.send`/`device.*`/`notifications.*`/`photos.latest`/`contacts.*`/`calendar.*`/`callLog.*`/`motion.*` を公開（[[sources/nodes/nodes]]）。各機能はランタイム権限が必要。
- Voice Wake は現状オフ（音声タブの手動マイク）。

## 設定・使い方の要点

- 自前ビルド：Java 17＋Android SDK（`./gradlew :app:assemblePlayDebug`）。デバイスペアリングを承認。

## 注意点・落とし穴

- 一般公開前。個人データコマンドは OS 権限プロンプトの承認が前提（拒否は `*_PERMISSION_REQUIRED`）。

## 用語と略称

- **モバイルノード** = スマホを Node として接続
- **個人データコマンド** = SMS/連絡先/カレンダー等の Android コマンドファミリー
- **ランタイム権限** = 実行時に付与する Android 権限

## 関連ページ

- [[components/node]] / [[sources/platforms/platforms]] / [[concepts/pairing]]
- [[sources/nodes/nodes]] / [[sources/nodes/location-command]]
