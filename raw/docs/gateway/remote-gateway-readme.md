---
title: "リモート Gateway のセットアップ"
source: "https://docs.openclaw.ai/ja-JP/gateway/remote-gateway-readme"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
> この内容は [リモートアクセス](https://docs.openclaw.ai/ja-JP/gateway/remote#macos-persistent-ssh-tunnel-via-launchagent) に統合されました。現在のガイドはそのページを参照してください。

## リモート Gateway で OpenClaw.app を実行する

OpenClaw.app は SSH トンネリングを使ってリモート Gateway に接続します。このガイドでは、その設定方法を示します。

## 概要

<svg id="oc_mermaid_1781443211714_0" width="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 338.65625px;" viewBox="0 0 338.65625 610" role="graphics-document document" aria-roledescription="flowchart-v2"><g><marker id="oc_mermaid_1781443211714_0_flowchart-v2-pointEnd" viewBox="0 0 10 10" refX="5" refY="5" markerUnits="userSpaceOnUse" markerWidth="8" markerHeight="8" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-pointStart" viewBox="0 0 10 10" refX="4.5" refY="5" markerUnits="userSpaceOnUse" markerWidth="8" markerHeight="8" orient="auto"><path d="M 0 5 L 10 10 L 10 0 z" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-pointEnd-margin" viewBox="0 0 11.5 14" refX="11.5" refY="7" markerUnits="userSpaceOnUse" markerWidth="10.5" markerHeight="14" orient="auto"><path d="M 0 0 L 11.5 7 L 0 14 z" style="stroke-width: 0; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-pointStart-margin" viewBox="0 0 11.5 14" refX="1" refY="7" markerUnits="userSpaceOnUse" markerWidth="11.5" markerHeight="14" orient="auto"><polygon points="0,7 11.5,14 11.5,0" style="stroke-width: 0; stroke-dasharray: 1, 0;"></polygon></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-circleEnd" viewBox="0 0 10 10" refX="11" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-circleStart" viewBox="0 0 10 10" refX="-1" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-circleEnd-margin" viewBox="0 0 10 10" refY="5" refX="12.25" markerUnits="userSpaceOnUse" markerWidth="14" markerHeight="14" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 0; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-circleStart-margin" viewBox="0 0 10 10" refX="-2" refY="5" markerUnits="userSpaceOnUse" markerWidth="14" markerHeight="14" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 0; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-crossEnd" viewBox="0 0 11 11" refX="12" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-crossStart" viewBox="0 0 11 11" refX="-1" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-crossEnd-margin" viewBox="0 0 15 15" refX="17.7" refY="7.5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 1,1 L 14,14 M 1,14 L 14,1" style="stroke-width: 2.5;"></path></marker><marker id="oc_mermaid_1781443211714_0_flowchart-v2-crossStart-margin" viewBox="0 0 15 15" refX="-3.5" refY="7.5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 1,1 L 14,14 M 1,14 L 14,1" style="stroke-width: 2.5; stroke-dasharray: 1, 0;"></path></marker><g><g><g id="oc_mermaid_1781443211714_0-Remote" data-look="classic"><rect style="" x="8" y="394" width="322.65625" height="208"></rect><g transform="translate(101.8984375, 394)"><foreignObject width="134.859375" height="24"><p>Remote Machine</p></foreignObject></g></g><g id="oc_mermaid_1781443211714_0-Client" data-look="classic"><rect style="" x="8" y="8" width="322.65625" height="336"></rect><g transform="translate(101.8984375, 8)"><foreignObject width="134.859375" height="24"><p>Client Machine</p></foreignObject></g></g></g><g><path d="M169.328,87L169.328,91.167C169.328,95.333,169.328,103.667,169.328,111.333C169.328,119,169.328,126,169.328,129.5L169.328,133" id="oc_mermaid_1781443211714_0-L_A_B_0" style=";" data-edge="true" data-et="edge" data-id="L_A_B_0" data-points="W3sieCI6MTY5LjMyODEyNSwieSI6ODd9LHsieCI6MTY5LjMyODEyNSwieSI6MTEyfSx7IngiOjE2OS4zMjgxMjUsInkiOjEzN31d" data-look="classic" marker-end="url(#oc_mermaid_1781443211714_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M169.328,215L169.328,219.167C169.328,223.333,169.328,231.667,169.328,239.333C169.328,247,169.328,254,169.328,257.5L169.328,261" id="oc_mermaid_1781443211714_0-L_B_T_0" style=";" data-edge="true" data-et="edge" data-id="L_B_T_0" data-points="W3sieCI6MTY5LjMyODEyNSwieSI6MjE1fSx7IngiOjE2OS4zMjgxMjUsInkiOjI0MH0seyJ4IjoxNjkuMzI4MTI1LCJ5IjoyNjV9XQ==" data-look="classic" marker-end="url(#oc_mermaid_1781443211714_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M169.328,473L169.328,477.167C169.328,481.333,169.328,489.667,169.328,497.333C169.328,505,169.328,512,169.328,515.5L169.328,519" id="oc_mermaid_1781443211714_0-L_C_D_0" style=";" data-edge="true" data-et="edge" data-id="L_C_D_0" data-points="W3sieCI6MTY5LjMyODEyNSwieSI6NDczfSx7IngiOjE2OS4zMjgxMjUsInkiOjQ5OH0seyJ4IjoxNjkuMzI4MTI1LCJ5Ijo1MjN9XQ==" data-look="classic" marker-end="url(#oc_mermaid_1781443211714_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M169.328,319L169.328,323.167C169.328,327.333,169.328,335.667,169.328,344C169.328,352.333,169.328,360.667,169.328,369C169.328,377.333,169.328,385.667,169.328,393.333C169.328,401,169.328,408,169.328,411.5L169.328,415" id="oc_mermaid_1781443211714_0-L_T_C_0" style=";" data-edge="true" data-et="edge" data-id="L_T_C_0" data-points="W3sieCI6MTY5LjMyODEyNSwieSI6MzE5fSx7IngiOjE2OS4zMjgxMjUsInkiOjM0NH0seyJ4IjoxNjkuMzI4MTI1LCJ5IjozNjl9LHsieCI6MTY5LjMyODEyNSwieSI6Mzk0fSx7IngiOjE2OS4zMjgxMjUsInkiOjQxOX1d" data-look="classic" marker-end="url(#oc_mermaid_1781443211714_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path></g><g><g><g data-id="L_A_B_0" transform="translate(0, 0)"></g></g><g><g data-id="L_B_T_0" transform="translate(0, 0)"></g></g><g><g data-id="L_C_D_0" transform="translate(0, 0)"></g></g><g><g data-id="L_T_C_0" transform="translate(0, 0)"></g></g></g><g><g id="oc_mermaid_1781443211714_0-flowchart-A-0" data-look="classic" transform="translate(169.328125, 60)"><rect style="" x="-87.796875" y="-27" width="175.59375" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-57.796875, -12)"><rect></rect><foreignObject width="115.59375" height="24"><p>OpenClaw.app</p></foreignObject></g></g><g id="oc_mermaid_1781443211714_0-flowchart-B-1" data-look="classic" transform="translate(169.328125, 176)"><rect style="" x="-126.328125" y="-39" width="252.65625" height="78" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-96.328125, -24)"><rect></rect><foreignObject width="192.65625" height="48"><p>ws://127.0.0.1:18789<br>(local port)</p></foreignObject></g></g><g id="oc_mermaid_1781443211714_0-flowchart-T-2" data-look="classic" transform="translate(169.328125, 292)"><rect style="" x="-78.1640625" y="-27" width="156.328125" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-48.1640625, -12)"><rect></rect><foreignObject width="96.328125" height="24"><p>SSH Tunnel</p></foreignObject></g></g><g id="oc_mermaid_1781443211714_0-flowchart-C-7" data-look="classic" transform="translate(169.328125, 446)"><rect style="" x="-111.8828125" y="-27" width="223.765625" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-81.8828125, -12)"><rect></rect><foreignObject width="163.765625" height="24"><p>Gateway WebSocket</p></foreignObject></g></g><g id="oc_mermaid_1781443211714_0-flowchart-D-8" data-look="classic" transform="translate(169.328125, 550)"><rect style="" x="-126.328125" y="-27" width="252.65625" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-96.328125, -12)"><rect></rect><foreignObject width="192.65625" height="24"><p>ws://127.0.0.1:18789</p></foreignObject></g></g></g></g></g><defs></defs><defs></defs><linearGradient id="oc_mermaid_1781443211714_0-gradient" gradientUnits="objectBoundingBox" x1="0%" y1="0%" x2="100%" y2="0%"><stop offset="0%" stop-color="#cccccc" stop-opacity="1"></stop><stop offset="100%" stop-color="hsl(180, 0%, 18.3529411765%)" stop-opacity="1"></stop></linearGradient></svg>

## クイックセットアップ

### 手順 1: SSH 設定を追加する

`~/.ssh/config` を編集して、次を追加します。

ssh

```
Host remote-gateway
    HostName &lt;REMOTE_IP&gt;          # e.g., 172.27.187.184
    User &lt;REMOTE_USER&gt;            # e.g., jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

`&lt;REMOTE_IP&gt;` と `&lt;REMOTE_USER&gt;` を自分の値に置き換えてください。

### 手順 2: SSH キーをコピーする

公開鍵をリモートマシンにコピーします（一度だけパスワードを入力します）。

bash

```bash
ssh-copy-id -i ~/.ssh/id_rsa &lt;REMOTE_USER&gt;@&lt;REMOTE_IP&gt;
```

### 手順 3: リモート Gateway 認証を設定する

bash

```bash
openclaw config set gateway.remote.token "<your-token>"
```

リモート Gateway がパスワード認証を使う場合は、代わりに `gateway.remote.password` を使用します。 `OPENCLAW_GATEWAY_TOKEN` はシェルレベルの上書きとして引き続き有効ですが、永続的なリモートクライアント設定は `gateway.remote.token` / `gateway.remote.password` です。

### 手順 4: SSH トンネルを開始する

bash

```bash
ssh -N remote-gateway &
```

### 手順 5: OpenClaw.app を再起動する

bash

```bash
# Quit OpenClaw.app (⌘Q), then reopen:
open /path/to/OpenClaw.app
```

これでアプリは SSH トンネル経由でリモート Gateway に接続します。

---

## ログイン時にトンネルを自動開始する

ログイン時に SSH トンネルを自動的に開始するには、Launch Agent を作成します。

### PLIST ファイルを作成する

これを `~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist` として保存します。

xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### Launch Agent を読み込む

bash

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist
```

これでトンネルは次のようになります。

- ログイン時に自動的に開始する
- クラッシュした場合に再起動する
- バックグラウンドで実行し続ける

レガシーメモ: 残っている `com.openclaw.ssh-tunnel` LaunchAgent がある場合は削除してください。

---

## トラブルシューティング

**トンネルが実行中か確認する:**

bash

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**トンネルを再起動する:**

bash

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.ssh-tunnel
```

**トンネルを停止する:**

bash

```bash
launchctl bootout gui/$UID/ai.openclaw.ssh-tunnel
```

---

## 仕組み

| コンポーネント | 役割 |
| --- | --- |
| `LocalForward 18789 127.0.0.1:18789` | ローカルポート 18789 をリモートポート 18789 に転送する |
| `ssh -N` | リモートコマンドを実行しない SSH（ポート転送のみ） |
| `KeepAlive` | クラッシュした場合にトンネルを自動的に再起動する |
| `RunAtLoad` | エージェントの読み込み時にトンネルを開始する |

OpenClaw.app はクライアントマシン上の `ws://127.0.0.1:18789` に接続します。SSH トンネルは、その接続を Gateway が実行されているリモートマシンのポート 18789 に転送します。

## 関連

- [Tailscale](https://docs.openclaw.ai/ja-JP/gateway/tailscale)