---
title: "プラットフォーム"
source: "https://docs.openclaw.ai/ja-JP/platforms"
author:
published:
created: 2026-06-14
description: "プラットフォームサポート概要（Gateway + コンパニオンアプリ）"
tags:
  - "clippings"
---
OpenClaw コアは TypeScript で書かれています。 **Node が推奨ランタイムです** 。 Bun は Gateway には推奨されません。WhatsApp と Telegram チャネルで既知の問題があります。詳細は [Bun (実験的)](https://docs.openclaw.ai/ja-JP/install/bun) を参照してください。

macOS（メニューバーアプリ）とモバイルノード（iOS/Android）向けのコンパニオンアプリがあります。Windows と Linux のコンパニオンアプリは計画中ですが、Gateway は現在完全にサポートされています。 Windows 向けのネイティブコンパニオンアプリも計画中です。Gateway は WSL2 経由での利用が推奨されます。

## OS を選択する

- macOS: [macOS](https://docs.openclaw.ai/ja-JP/platforms/macos)
- iOS: [iOS](https://docs.openclaw.ai/ja-JP/platforms/ios)
- Android: [Android](https://docs.openclaw.ai/ja-JP/platforms/android)
- Windows: [Windows](https://docs.openclaw.ai/ja-JP/platforms/windows)
- Linux: [Linux](https://docs.openclaw.ai/ja-JP/platforms/linux)

## VPS とホスティング

- VPS ハブ: [VPS ホスティング](https://docs.openclaw.ai/ja-JP/vps)
- Fly.io: [Fly.io](https://docs.openclaw.ai/ja-JP/install/fly)
- Hetzner（Docker）: [Hetzner](https://docs.openclaw.ai/ja-JP/install/hetzner)
- GCP（Compute Engine）: [GCP](https://docs.openclaw.ai/ja-JP/install/gcp)
- Azure（Linux VM）: [Azure](https://docs.openclaw.ai/ja-JP/install/azure)
- exe.dev（VM + HTTPS プロキシ）: [exe.dev](https://docs.openclaw.ai/ja-JP/install/exe-dev)

## 共通リンク

- インストールガイド: [はじめに](https://docs.openclaw.ai/ja-JP/start/getting-started)
- Gateway ランブック: [Gateway](https://docs.openclaw.ai/ja-JP/gateway)
- Gateway 設定: [設定](https://docs.openclaw.ai/ja-JP/gateway/configuration)
- サービスステータス: `openclaw gateway status`

## Gateway サービスのインストール（CLI）

次のいずれかを使用します（すべてサポートされています）。

- ウィザード（推奨）: `openclaw onboard --install-daemon`
- 直接: `openclaw gateway install`
- 設定フロー: `openclaw configure` → **Gateway サービス** を選択
- 修復/移行: `openclaw doctor` （サービスのインストールまたは修復を提案します）

サービスターゲットは OS によって異なります。

- macOS: LaunchAgent（ `ai.openclaw.gateway` または `ai.openclaw.<profile>` 、レガシーは `com.openclaw.*` ）
- Linux/WSL2: systemd ユーザーサービス（ `openclaw-gateway[-<profile>].service` ）
- ネイティブ Windows: スケジュールされたタスク（ `OpenClaw Gateway` または `OpenClaw Gateway (<profile>)` ）。タスク作成が拒否された場合は、ユーザーごとの Startup フォルダーのログイン項目にフォールバックします

## 関連

- [インストール概要](https://docs.openclaw.ai/ja-JP/install)
- [macOS アプリ](https://docs.openclaw.ai/ja-JP/platforms/macos)
- [iOS アプリ](https://docs.openclaw.ai/ja-JP/platforms/ios)