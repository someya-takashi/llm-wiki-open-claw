---
type: source
source_kind: docs
source_url: https://docs.openclaw.ai/ja-JP/security/network-proxy
source_path: raw/docs/security/network-proxy.md
doc_section: security
title: "ネットワークプロキシ"
ingested: 2026-06-14
tags: [network-proxy, ssrf, outbound, forward-proxy, hardening, egress]
related:
  - "[[concepts/security]]"
  - "[[concepts/sandboxing]]"
  - "[[components/gateway]]"
---

# ネットワークプロキシ（解説）

> 原典: `raw/docs/security/network-proxy.md` ・ https://docs.openclaw.ai/ja-JP/security/network-proxy

## 一言まとめ

OpenClaw のランタイム HTTP/WebSocket 送信トラフィックを、**オペレーター管理のフォワードプロキシ**経由にルーティングする任意の多層防御。中央集権的な送信制御・より強い SSRF（Server-Side Request Forgery, サーバーを踏み台に内部へ送らせる攻撃）保護・監査性を狙う。

## 位置づけ

[[concepts/security]] の送信側ハードニング（[[concepts/sandboxing]] と並ぶプロセスレベルのガードレール）。OpenClaw はプロキシを同梱/起動/認定せず、ユーザーが用意したプロキシに通常の HTTP/WS クライアントをルーティングするだけ。

## 仕組み・ふるまい

- `proxy.enabled=true`＋プロキシ URL で、`openclaw gateway run`/`node run`/`agent --local` 等の保護プロセスが `fetch`/`node:http(s)`/WebSocket をプロキシ経由に。内部は Undici ディスパッチャー＋ `global-agent` の 2 フック。
- **接続時チェック**：DNS 解決後、上流接続直前に宛先を評価（DNS リバインディング防御）。`no_proxy`/`NO_PROXY` 等はクリア（宛先ベースの抜け穴を塞ぐ）。
- **Gateway ループバック**：`proxy.loopbackMode`（`gateway-only`既定=ローカル Gateway WS は直接／`proxy`=プロキシ経由／`block`=拒否）。

## 設定・使い方の要点

- 設定：`proxy: { enabled: true, proxyUrl: "http://127.0.0.1:3128" }`（`proxy.proxyUrl` > `OPENCLAW_PROXY_URL`）。プロキシ URL は `http://`（HTTPS 宛先は CONNECT トンネル）。
- 検証：`openclaw proxy validate --proxy-url ...`（`example.com` 成功＋ループバックカナリア拒否を確認、`--json`/`--apns-reachable`）。
- ⚠️ `enabled=true` でプロキシ URL 未設定だと、保護コマンドは直接接続にフォールバックせず**起動失敗**（フェイルクローズ）。

## 注意点・落とし穴

- **プロキシポリシーがセキュリティ境界**：OpenClaw は正しくブロックしているか検証できない。プロキシは loopback/private のみbind、宛先を自前解決、ループバック/private/リンクローカル/メタデータ（`169.254.169.254` 等）/予約範囲をブロック（推奨拒否リストあり、参照は `src/infra/net/ssrf.ts`）。
- OS レベルのサンドボックスではない。生 `net`/`tls`/`http2`、ネイティブアドオン、IRC（生 TCP/TLS、`channels.irc.enabled=false` 推奨）は迂回し得る。

## 用語と略称

- **フォワードプロキシ（forward proxy）** = 送信トラフィックを集約する出口プロキシ
- **SSRF** = Server-Side Request Forgery
- **DNS リバインディング** = DNS 応答を後で差し替えて内部に到達する攻撃
- **CONNECT トンネル** = HTTP プロキシ越しに TLS を張る方式
- **フェイルクローズ（fail closed）** = 不備時に通すのでなく塞ぐ安全側の挙動

## 関連ページ

- [[concepts/security]] / [[concepts/sandboxing]] / [[concepts/threat-model]]
- [[sources/gateway/trusted-proxy-auth]] / [[components/gateway]]
