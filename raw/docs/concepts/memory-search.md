---
title: "メモリ検索"
source: "https://docs.openclaw.ai/ja-JP/concepts/memory-search"
author:
published:
created: 2026-06-14
description: "OpenClaw は、あらゆる OS で動作する AI エージェント向けのマルチチャネルGatewayです。"
tags:
  - "clippings"
---
`memory_search` は、表現が元のテキストと異なる場合でも、メモリファイルから関連するノートを見つけます。メモリを小さなチャンクにインデックス化し、埋め込み、キーワード、またはその両方を使って検索します。

## クイックスタート

GitHub Copilot サブスクリプション、OpenAI、Gemini、Voyage、または Mistral API キーを設定している場合、メモリ検索は自動的に動作します。プロバイダーを明示的に設定するには:

json5

```
{
  agents: {
    defaults: {
      memorySearch: {
        provider: "openai", // or "gemini", "local", "ollama", etc.
      },
    },
  },
}
```

複数エンドポイント構成では、 `provider` に `ollama-5080` などのカスタム `models.providers.<id>` エントリも指定できます。そのプロバイダーが `api: "ollama"` または別の埋め込みアダプター所有者を設定している場合です。

API キーなしのローカル埋め込みには、 `provider: "local"` を設定します。ソースチェックアウトでは、引き続きネイティブビルドの承認が必要な場合があります: `pnpm approve-builds` の後に `pnpm rebuild node-llama-cpp` を実行します。

一部の OpenAI 互換埋め込みエンドポイントでは、検索には `input_type: "query"` 、インデックス化されたチャンクには `input_type: "document"` または `"passage"` のような非対称ラベルが必要です。これらは `memorySearch.queryInputType` と `memorySearch.documentInputType` で設定します。詳しくは [メモリ設定リファレンス](https://docs.openclaw.ai/ja-JP/reference/memory-config#provider-specific-config) を参照してください。

## 対応プロバイダー

| プロバイダー | ID | API キーが必要 | 注記 |
| --- | --- | --- | --- |
| Bedrock | `bedrock` | いいえ | AWS 認証情報チェーンが解決されると自動検出されます |
| Gemini | `gemini` | はい | 画像/音声のインデックス化に対応 |
| GitHub Copilot | `github-copilot` | いいえ | 自動検出され、Copilot サブスクリプションを使用 |
| Local | `local` | いいえ | GGUF モデル、約 0.6 GB のダウンロード |
| Mistral | `mistral` | はい | 自動検出 |
| Ollama | `ollama` | いいえ | ローカル。明示的に設定する必要があります |
| OpenAI | `openai` | はい | 自動検出、高速 |
| Voyage | `voyage` | はい | 自動検出 |

## 検索の仕組み

OpenClaw は 2 つの取得経路を並列に実行し、結果をマージします:

<svg id="oc_mermaid_1781437281035_0" width="100%" xmlns="http://www.w3.org/2000/svg" style="max-width: 1016.9375px;" viewBox="0 0 1016.9375 174" role="graphics-document document" aria-roledescription="flowchart-v2"><g><marker id="oc_mermaid_1781437281035_0_flowchart-v2-pointEnd" viewBox="0 0 10 10" refX="5" refY="5" markerUnits="userSpaceOnUse" markerWidth="8" markerHeight="8" orient="auto"><path d="M 0 0 L 10 5 L 0 10 z" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-pointStart" viewBox="0 0 10 10" refX="4.5" refY="5" markerUnits="userSpaceOnUse" markerWidth="8" markerHeight="8" orient="auto"><path d="M 0 5 L 10 10 L 10 0 z" style="stroke-width: 1; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-pointEnd-margin" viewBox="0 0 11.5 14" refX="11.5" refY="7" markerUnits="userSpaceOnUse" markerWidth="10.5" markerHeight="14" orient="auto"><path d="M 0 0 L 11.5 7 L 0 14 z" style="stroke-width: 0; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-pointStart-margin" viewBox="0 0 11.5 14" refX="1" refY="7" markerUnits="userSpaceOnUse" markerWidth="11.5" markerHeight="14" orient="auto"><polygon points="0,7 11.5,14 11.5,0" style="stroke-width: 0; stroke-dasharray: 1, 0;"></polygon></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-circleEnd" viewBox="0 0 10 10" refX="11" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-circleStart" viewBox="0 0 10 10" refX="-1" refY="5" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 1; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-circleEnd-margin" viewBox="0 0 10 10" refY="5" refX="12.25" markerUnits="userSpaceOnUse" markerWidth="14" markerHeight="14" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 0; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-circleStart-margin" viewBox="0 0 10 10" refX="-2" refY="5" markerUnits="userSpaceOnUse" markerWidth="14" markerHeight="14" orient="auto"><circle cx="5" cy="5" r="5" style="stroke-width: 0; stroke-dasharray: 1, 0;"></circle></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-crossEnd" viewBox="0 0 11 11" refX="12" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-crossStart" viewBox="0 0 11 11" refX="-1" refY="5.2" markerUnits="userSpaceOnUse" markerWidth="11" markerHeight="11" orient="auto"><path d="M 1,1 l 9,9 M 10,1 l -9,9" style="stroke-width: 2; stroke-dasharray: 1, 0;"></path></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-crossEnd-margin" viewBox="0 0 15 15" refX="17.7" refY="7.5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 1,1 L 14,14 M 1,14 L 14,1" style="stroke-width: 2.5;"></path></marker><marker id="oc_mermaid_1781437281035_0_flowchart-v2-crossStart-margin" viewBox="0 0 15 15" refX="-3.5" refY="7.5" markerUnits="userSpaceOnUse" markerWidth="12" markerHeight="12" orient="auto"><path d="M 1,1 L 14,14 M 1,14 L 14,1" style="stroke-width: 2.5; stroke-dasharray: 1, 0;"></path></marker><g><g></g><g><path d="M103.15,60L109.487,55.833C115.824,51.667,128.498,43.333,138.335,39.167C148.172,35,155.172,35,158.672,35L162.172,35" id="oc_mermaid_1781437281035_0-L_Q_E_0" style=";" data-edge="true" data-et="edge" data-id="L_Q_E_0" data-points="W3sieCI6MTAzLjE0OTc4OTY2MzQ2MTU1LCJ5Ijo2MH0seyJ4IjoxNDEuMTcxODc1LCJ5IjozNX0seyJ4IjoxNjYuMTcxODc1LCJ5IjozNX1d" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M103.15,114L109.487,118.167C115.824,122.333,128.498,130.667,139.138,134.833C149.779,139,158.385,139,162.689,139L166.992,139" id="oc_mermaid_1781437281035_0-L_Q_T_0" style=";" data-edge="true" data-et="edge" data-id="L_Q_T_0" data-points="W3sieCI6MTAzLjE0OTc4OTY2MzQ2MTU1LCJ5IjoxMTR9LHsieCI6MTQxLjE3MTg3NSwieSI6MTM5fSx7IngiOjE3MC45OTIxODc1LCJ5IjoxMzl9XQ==" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M312.875,35L317.042,35C321.208,35,329.542,35,337.208,35C344.875,35,351.875,35,355.375,35L358.875,35" id="oc_mermaid_1781437281035_0-L_E_VS_0" style=";" data-edge="true" data-et="edge" data-id="L_E_VS_0" data-points="W3sieCI6MzEyLjg3NSwieSI6MzV9LHsieCI6MzM3Ljg3NSwieSI6MzV9LHsieCI6MzYyLjg3NSwieSI6MzV9XQ==" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M308.055,139L313.025,139C317.995,139,327.935,139,338.01,139C348.086,139,358.297,139,363.402,139L368.508,139" id="oc_mermaid_1781437281035_0-L_T_BM_0" style=";" data-edge="true" data-et="edge" data-id="L_T_BM_0" data-points="W3sieCI6MzA4LjA1NDY4NzUsInkiOjEzOX0seyJ4IjozMzcuODc1LCJ5IjoxMzl9LHsieCI6MzcyLjUwNzgxMjUsInkiOjEzOX1d" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M548.109,35L552.276,35C556.443,35,564.776,35,578.139,38.906C591.502,42.812,609.895,50.624,619.092,54.53L628.288,58.436" id="oc_mermaid_1781437281035_0-L_VS_M_0" style=";" data-edge="true" data-et="edge" data-id="L_VS_M_0" data-points="W3sieCI6NTQ4LjEwOTM3NSwieSI6MzV9LHsieCI6NTczLjEwOTM3NSwieSI6MzV9LHsieCI6NjMxLjk2OTgwMTY4MjY5MjMsInkiOjYwfV0=" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M538.477,139L544.249,139C550.021,139,561.565,139,576.534,135.094C591.502,131.188,609.895,123.376,619.092,119.47L628.288,115.564" id="oc_mermaid_1781437281035_0-L_BM_M_0" style=";" data-edge="true" data-et="edge" data-id="L_BM_M_0" data-points="W3sieCI6NTM4LjQ3NjU2MjUsInkiOjEzOX0seyJ4Ijo1NzMuMTA5Mzc1LCJ5IjoxMzl9LHsieCI6NjMxLjk2OTgwMTY4MjY5MjMsInkiOjExNH1d" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path><path d="M792.969,87L797.135,87C801.302,87,809.635,87,817.302,87C824.969,87,831.969,87,835.469,87L838.969,87" id="oc_mermaid_1781437281035_0-L_M_R_0" style=";" data-edge="true" data-et="edge" data-id="L_M_R_0" data-points="W3sieCI6NzkyLjk2ODc1LCJ5Ijo4N30seyJ4Ijo4MTcuOTY4NzUsInkiOjg3fSx7IngiOjg0Mi45Njg3NSwieSI6ODd9XQ==" data-look="classic" marker-end="url(#oc_mermaid_1781437281035_0_flowchart-v2-pointEnd)" fill="none" stroke="currentColor"></path></g><g><g><g data-id="L_Q_E_0" transform="translate(0, 0)"></g></g><g><g data-id="L_Q_T_0" transform="translate(0, 0)"></g></g><g><g data-id="L_E_VS_0" transform="translate(0, 0)"></g></g><g><g data-id="L_T_BM_0" transform="translate(0, 0)"></g></g><g><g data-id="L_VS_M_0" transform="translate(0, 0)"></g></g><g><g data-id="L_BM_M_0" transform="translate(0, 0)"></g></g><g><g data-id="L_M_R_0" transform="translate(0, 0)"></g></g></g><g><g id="oc_mermaid_1781437281035_0-flowchart-Q-0" data-look="classic" transform="translate(62.0859375, 87)"><rect style="" x="-54.0859375" y="-27" width="108.171875" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-24.0859375, -12)"><rect></rect><foreignObject width="48.171875" height="24"><p>Query</p></foreignObject></g></g><g id="oc_mermaid_1781437281035_0-flowchart-E-1" data-look="classic" transform="translate(239.5234375, 35)"><rect style="" x="-73.3515625" y="-27" width="146.703125" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-43.3515625, -12)"><rect></rect><foreignObject width="86.703125" height="24"><p>Embedding</p></foreignObject></g></g><g id="oc_mermaid_1781437281035_0-flowchart-T-3" data-look="classic" transform="translate(239.5234375, 139)"><rect style="" x="-68.53125" y="-27" width="137.0625" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-38.53125, -12)"><rect></rect><foreignObject width="77.0625" height="24"><p>Tokenize</p></foreignObject></g></g><g id="oc_mermaid_1781437281035_0-flowchart-VS-5" data-look="classic" transform="translate(455.4921875, 35)"><rect style="" x="-92.6171875" y="-27" width="185.234375" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-62.6171875, -12)"><rect></rect><foreignObject width="125.234375" height="24"><p>Vector Search</p></foreignObject></g></g><g id="oc_mermaid_1781437281035_0-flowchart-BM-7" data-look="classic" transform="translate(455.4921875, 139)"><rect style="" x="-82.984375" y="-27" width="165.96875" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-52.984375, -12)"><rect></rect><foreignObject width="105.96875" height="24"><p>BM25 Search</p></foreignObject></g></g><g id="oc_mermaid_1781437281035_0-flowchart-M-9" data-look="classic" transform="translate(695.5390625, 87)"><rect style="" x="-97.4296875" y="-27" width="194.859375" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-67.4296875, -12)"><rect></rect><foreignObject width="134.859375" height="24"><p>Weighted Merge</p></foreignObject></g></g><g id="oc_mermaid_1781437281035_0-flowchart-R-13" data-look="classic" transform="translate(925.953125, 87)"><rect style="" x="-82.984375" y="-27" width="165.96875" height="54" fill="none" stroke="currentColor"></rect><g style="" transform="translate(-52.984375, -12)"><rect></rect><foreignObject width="105.96875" height="24"><p>Top Results</p></foreignObject></g></g></g></g></g><defs></defs><defs></defs><linearGradient id="oc_mermaid_1781437281035_0-gradient" gradientUnits="objectBoundingBox" x1="0%" y1="0%" x2="100%" y2="0%"><stop offset="0%" stop-color="#cccccc" stop-opacity="1"></stop><stop offset="100%" stop-color="hsl(180, 0%, 18.3529411765%)" stop-opacity="1"></stop></linearGradient></svg>
- **ベクトル検索** は、意味が似ているノートを見つけます（"gateway host" は "the machine running OpenClaw" に一致します）。
- **BM25 キーワード検索** は、完全一致を見つけます（ID、エラー文字列、設定キー）。

片方の経路だけが利用可能な場合（埋め込みがない、または FTS がない）、もう片方だけが実行されます。

埋め込みが利用できない場合でも、OpenClaw は未加工の完全一致順序のみにフォールバックするのではなく、FTS 結果に対して字句ランキングを使用します。この縮退モードでは、クエリ語のカバレッジが強く、関連するファイルパスを持つチャンクがブーストされるため、 `sqlite-vec` や埋め込みプロバイダーがなくても有用な再現率を維持できます。

## 検索品質の改善

ノート履歴が大きい場合に役立つ 2 つの任意機能があります:

### 時間減衰

古いノートはランキングの重みを徐々に失うため、最近の情報が先に表示されます。 デフォルトの半減期 30 日では、先月のノートは元の重みの 50% でスコアリングされます。 `MEMORY.md` のような常緑ファイルは減衰されません。

> [!note] Note
> **Tip**
> 
> エージェントに数か月分の日次ノートがあり、古い情報が最近のコンテキストより上位に出続ける場合は、時間減衰を有効にしてください。

### MMR（多様性）

重複した結果を減らします。5 つのノートがすべて同じルーター設定に言及している場合、MMR により、上位結果が繰り返しではなく異なるトピックを網羅するようになります。

> [!note] Note
> **Tip**
> 
> `memory_search` が異なる日次ノートからほぼ重複したスニペットを返し続ける場合は、MMR を有効にしてください。

### 両方を有効にする

json5

```
{
  agents: {
    defaults: {
      memorySearch: {
        query: {
          hybrid: {
            mmr: { enabled: true },
            temporalDecay: { enabled: true },
          },
        },
      },
    },
  },
}
```

## マルチモーダルメモリ

Gemini Embedding 2 を使用すると、Markdown と一緒に画像や音声ファイルをインデックス化できます。検索クエリはテキストのままですが、視覚および音声コンテンツに照合されます。設定については [メモリ設定リファレンス](https://docs.openclaw.ai/ja-JP/reference/memory-config) を参照してください。

## セッションメモリ検索

任意でセッショントランスクリプトをインデックス化し、 `memory_search` が以前の会話を思い出せるようにできます。これは `memorySearch.experimental.sessionMemory` によるオプトインです。詳しくは [設定リファレンス](https://docs.openclaw.ai/ja-JP/reference/memory-config) を参照してください。

## トラブルシューティング

**結果がありませんか?** インデックスを確認するには `openclaw memory status` を実行します。空の場合は、 `openclaw memory index --force` を実行します。

**キーワード一致のみですか?** 埋め込みプロバイダーが設定されていない可能性があります。 `openclaw memory status --deep` を確認してください。

**ローカル埋め込みがタイムアウトしますか?** `ollama` 、 `lmstudio` 、 `local` はデフォルトで長めのインラインバッチタイムアウトを使用します。ホストが単に遅い場合は、 `agents.defaults.memorySearch.sync.embeddingBatchTimeoutSeconds` を設定し、 `openclaw memory index --force` を再実行します。

**CJK テキストが見つかりませんか?** FTS インデックスを `openclaw memory index --force` で再構築してください。

## 参考資料

- [Active Memory](https://docs.openclaw.ai/ja-JP/concepts/active-memory) -- 対話型チャットセッション用のサブエージェントメモリ
- [メモリ](https://docs.openclaw.ai/ja-JP/concepts/memory) -- ファイルレイアウト、バックエンド、ツール
- [メモリ設定リファレンス](https://docs.openclaw.ai/ja-JP/reference/memory-config) -- すべての設定ノブ

## 関連

- [Active Memory](https://docs.openclaw.ai/ja-JP/concepts/active-memory)
- [組み込みメモリエンジン](https://docs.openclaw.ai/ja-JP/concepts/memory-builtin)