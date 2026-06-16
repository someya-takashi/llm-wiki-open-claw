---
type: translation
of: "[[articles/context-engineering-session-memory]]"
source_url: https://developers.openai.com/cookbook/examples/agents_sdk/session_memory
lang: ja
translated: 2026-06-15
---

# コンテキストエンジニアリング — Session による短期メモリ管理

> 原典: `raw/articles/Context Engineering - Short-Term Memory Management with Sessions.md` ・ https://developers.openai.com/cookbook/examples/agents_sdk/session_memory
> 要約は [[articles/context-engineering-session-memory]]

AI エージェントはしばしば**長時間にわたる複数ターンの対話**で動作し、そこでは**コンテキスト**の適切なバランスを保つことが極めて重要です。持ち越す情報が多すぎると、モデルは注意散漫・非効率、あるいは完全な失敗のリスクを負います。保持する情報が少なすぎると、エージェントは一貫性を失います。

ここでいうコンテキストとは、モデルが一度に注意を向けられるトークンの総ウィンドウ（入力＋出力）を指します。 [GPT-5](https://platform.openai.com/docs/models/gpt-5) では、この容量は入力で最大 272k トークン、出力で 128k トークンに達しますが、そのような大きなウィンドウであっても、精選されていない履歴・冗長なツール結果・ノイズの多い検索結果によって容易に飽和してしまいます。このため、コンテキスト管理は単なる最適化ではなく、必須事項となります。

本クックブックでは、 [OpenAI Agents SDK](https://github.com/openai/openai-agents-python) の `Session` オブジェクトを使って**コンテキストを効果的に管理する方法**を探ります。エージェントを高速・信頼性・コスト効率の高い状態に保つために、実証済みの 2 つのコンテキスト管理技法――**トリミング（trimming）**と**圧縮（compression）**――に焦点を当てます。

#### なぜコンテキスト管理が重要なのか

- **長いスレッド全体での一貫性の維持** – 古い詳細を引きずることなく、エージェントを最新のユーザー目標に固定し続けます。セッションレベルのトリミングと要約により、「昨日の計画」が「今日の依頼」を上書きしてしまうことを防ぎます。
- **より高いツール呼び出し精度** – 焦点の絞られたコンテキストは関数選択と引数の充填を改善し、複数ツール実行中の再試行・タイムアウト・連鎖的失敗を減らします。
- **より低いレイテンシとコスト** – より小さく鋭いプロンプトは、1 ターンあたりのトークン数と注意の負荷を削減します。
- **エラーと幻覚の封じ込め** – 要約は、過去の誤りを訂正または省略する「クリーンルーム」として機能します。トリミングは、ターンごとに悪い事実を増幅すること（「コンテキストポイズニング（context poisoning）」）を回避します。
- **より容易なデバッグと可観測性** – 安定した要約と境界の定まった履歴は、ログを比較可能にします。要約を差分比較し、回帰の原因を特定し、失敗を確実に再現できます。
- **複数課題とハンドオフへの耐性** – 複数の問題を含むチャットでは、課題ごとのミニ要約により、エージェントは一貫性を保ちながら一時停止／再開したり、人間へエスカレーションしたり、別のエージェントへ引き継いだりできます。

![AIエージェントにおけるメモリの比較](https://developers.openai.com/cookbook/assets/images/memory_comparison.jpg)

[OpenAI Responses API](https://platform.openai.com/docs/api-reference/responses/create#responses-create-previous_response_id) には、組み込みの状態管理と `previous_response_id` によるメッセージの連結を通じた**基本的なメモリサポート**が含まれています。

直前のレスポンスの `id` を `previous_response_id` として渡すことで会話を継続できます。あるいは、出力をリストに集めて次のレスポンスの `input` として再送信することで、手動でコンテキストを管理することもできます。

そこで得られないのが、**自動的なメモリ管理**です。そこで **Agents SDK** の出番となります。これは Responses の上に [セッションメモリ](https://openai.github.io/openai-agents-python/sessions/) を提供するので、`response.output` を手動で追加したり、自分で ID を追跡したりする必要がなくなります。セッションが**メモリオブジェクト**になります。`session.run("...")` を繰り返し呼び出すだけで、SDK がコンテキスト長・履歴・継続性を処理してくれるため、一貫した複数ターンのエージェントを構築することがはるかに容易になります。

#### 実世界のシナリオ

これらの技法を、長時間タスクのよくある一例で具体化します。例えば次のようなものです：

- **複数ターンのカスタマーサービス会話** ハードウェアとソフトウェアの両方にまたがる技術製品についての長い会話では、顧客はしばしば時間をかけて複数の問題を表面化させます。エージェントは、過去のあらゆる詳細を引きずるのではなく、要点だけを保持しながら、一貫性と目標への集中を保たなければなりません。

#### 扱う技法

これらの課題に対処するため、OpenAI Agents SDK を用いた 2 つの別個の具体的アプローチを紹介します：

- **コンテキストトリミング（Context Trimming）** – 直近 N ターンを保持しつつ、古いターンを破棄する。
	- **長所**
		- **決定論的かつシンプル：** 要約器のばらつきがない。状態の推論や実行の再現が容易。
				- **追加レイテンシゼロ：** 履歴を圧縮するための追加のモデル呼び出しがない。
				- **直近作業への忠実性：** 最新のツール結果・パラメータ・エッジケースがそのまま（verbatim）残る――デバッグに最適。
				- **「要約ドリフト」のリスクが低い：** 事実を再解釈したり圧縮したりすることがない。
		**短所**
		- **長距離コンテキストを唐突に忘れる：** 重要な以前の制約・ID・決定が、N を過ぎてスクロールアウトすると消えうる。
				- **ユーザー体験上の「健忘」：** 長いセッションの途中で、エージェントが約束や以前の好みを「忘れた」ように見えることがある。
				- **シグナルの浪費：** 古いターンには再利用可能な知識（要件・制約）が含まれていることがあり、それが破棄される。
				- **トークンスパイクは依然として起こりうる：** 直近のターンに巨大なツールペイロードが含まれていれば、last-N でもコンテキストが膨れ上がりうる。
		- **適している場面**
		- 会話中のタスクが互いに独立しており、コンテキストが重複せず、以前の詳細をさらに持ち越す必要がない場合。
				- 予測可能性・容易な評価・低レイテンシが必要な場合（運用自動化、CRM/API アクション）。
				- 会話の有用なコンテキストが局所的な場合（遠い履歴より直近のステップがはるかに重要）。
- **コンテキスト要約（Context Summarization）** – 以前のメッセージ（assistant・user・tools 等）を、構造化された短い要約に圧縮し、会話履歴に注入する。
	- **長所**
		- **長距離メモリをコンパクトに保持：** 過去の要件・決定・根拠が N を超えて持続する。
				- **より滑らかな UX：** エージェントが長いセッションを通じて約束や制約を「覚えている」。
				- **コストを抑えたスケール：** 1 つの簡潔な要約が数百ターンを置き換えうる。
				- **検索可能なアンカー：** 1 つの合成 assistant メッセージが、安定した「これまでの世界の状態」になる。
		**短所**
		- **要約の損失とバイアス：** 詳細が落ちたり重み付けを誤ったりしうる。微妙な制約が消えることがある。
				- **レイテンシとコストのスパイク：** リフレッシュのたびにモデル処理が増える（場合によってはツールトリムのロジックも）。
				- **誤りの複利的増大：** 悪い事実が要約に入ると、将来の挙動を**汚染（poison）**しうる（「コンテキストポイズニング」）。
				- **可観測性の複雑化：** 監査と評価のために、要約のプロンプト/出力をログに残さねばならない。
		- **適している場面**
		- 計画立案/コーチング、RAG 中心の分析、ポリシー Q&A のように、フロー全体にわたって収集されたコンテキストをタスクが必要とするユースケース。
				- 長い時間軸での継続性が必要で、関連タスクを解くために重要な詳細をさらに持ち越す場合。
				- セッションが N ターンを超えるが、決定・ID・制約を確実に保持しなければならない場合。

**簡易比較**

| 観点 | **トリミング（last-N ターン）** | **要約（古いもの → 生成された要約）** |
| --- | --- | --- |
| レイテンシ / コスト | 最低（追加呼び出しなし） | 要約リフレッシュ時点で高くなる |
| 長距離の想起 | 弱い（ハードな打ち切り） | 強い（コンパクトな持ち越し） |
| リスクの種類 | コンテキストの喪失 | コンテキストの歪み/ポイズニング |
| 可観測性 | シンプルなログ | 要約のプロンプト/出力をログ必須 |
| 評価の安定性 | 高い | 堅牢な要約評価が必要 |
| 最適な用途 | ツール中心の運用、短いワークフロー | アナリスト/コンシェルジュ、長いスレッド |

## 前提条件

このクックブックを実行する前に、以下のアカウントを設定し、いくつかのセットアップ作業を完了する必要があります。これらの前提条件は、本プロジェクトで使用する API とやり取りするために不可欠です。

#### ステップ0：OpenAI アカウントと OPENAI\_API\_KEY

- **目的：**  
	言語モデルにアクセスし、本クックブックで取り上げる Agents SDK を使用するには、OpenAI アカウントが必要です。
- **アクション：**  
	まだ持っていない場合は [OpenAI アカウントにサインアップ](https://openai.com/) してください。アカウントを取得したら、 [OpenAI API Keys ページ](https://platform.openai.com/api-keys) にアクセスして API キーを作成します。

**ワークフローを実行する前に、環境変数を設定してください：**

```plaintext
# Your openai key
os.environ["OPENAI_API_KEY"] = "sk-proj-..."
```

あるいは、agents ライブラリをインポートして `set_default_openai_key` 関数を使い、エージェントが使用する OpenAI API キーを設定することもできます。

```plaintext
from agents import set_default_openai_key
set_default_openai_key("YOUR_API_KEY")
```

#### ステップ1：必要なライブラリのインストール

以下で `openai-agents` ライブラリ（ [OpenAI Agents SDK](https://github.com/openai/openai-agents-python) ）をインストールします。

```python
%pip install openai-agents nest_asyncio
```
```python
from openai import OpenAI

client = OpenAI()
```
```python
from agents import set_tracing_disabled
set_tracing_disabled(True)
```

エージェントを定義して実行することで、インストールしたライブラリをテストしてみましょう。

```python
import asyncio
from agents import Agent, Runner

agent = Agent(
    name="Assistant",
    instructions="Reply very concisely.",
)

result = await Runner.run(agent, "Tell me why it is important to evaluate AI agents.")
print(result.final_output)
```
```text
Evaluating AI agents ensures reliability, safety, ethical alignment, performance accuracy, and helps avoid biases, improving overall trust and effectiveness.
```

### エージェントの定義

まず Agents SDK ライブラリから必要なコンポーネントを定義することから始められます。ユースケースに応じて、エージェント作成時に instructions を追加します。

#### カスタマーサービスエージェント

```python
support_agent = Agent(
    name="Customer Support Assistant",
    model="gpt-5",
    instructions=(
        "You are a patient, step-by-step IT support assistant. "
        "Your role is to help customers troubleshoot and resolve issues with devices and software. "
        "Guidelines:\n"
        "- Be concise and use numbered steps where possible.\n"
        "- Ask only one focused, clarifying question at a time before suggesting next actions.\n"
        "- Track and remember multiple issues across the conversation; update your understanding as new problems emerge.\n"
        "- When a problem is resolved, briefly confirm closure before moving to the next.\n"
    )
)
```

## コンテキストトリミング

#### カスタム Session オブジェクトの実装

[OpenAI Agents Python SDK](https://openai.github.io/openai-agents-python/) の [Session](https://openai.github.io/openai-agents-python/sessions/) オブジェクトを使用します。以下は、**直近 N ターンのみを保持する** `TrimmingSession` の実装です（「ターン」＝ 1 つの user メッセージと、次の user メッセージまでのすべて――assistant の返信や任意のツール呼び出し/結果を含む）。これはインメモリであり、すべての書き込みと読み取りで自動的にトリミングします。

```python
from __future__ import annotations

import asyncio
from collections import deque
from typing import Any, Deque, Dict, List, cast

from agents.memory.session import SessionABC
from agents.items import TResponseInputItem  # dict-like item

ROLE_USER = "user"

def _is_user_msg(item: TResponseInputItem) -> bool:
    """Return True if the item represents a user message."""
    # Common dict-shaped messages
    if isinstance(item, dict):
        role = item.get("role")
        if role is not None:
            return role == ROLE_USER
        # Some SDKs: {"type": "message", "role": "..."}
        if item.get("type") == "message":
            return item.get("role") == ROLE_USER
    # Fallback: objects with a .role attr
    return getattr(item, "role", None) == ROLE_USER

class TrimmingSession(SessionABC):
    """
    Keep only the last N *user turns* in memory.

    A turn = a user message and all subsequent items (assistant/tool calls/results)
    up to (but not including) the next user message.
    """

    def __init__(self, session_id: str, max_turns: int = 8):
        self.session_id = session_id
        self.max_turns = max(1, int(max_turns))
        self._items: Deque[TResponseInputItem] = deque()  # chronological log
        self._lock = asyncio.Lock()

    # ---- SessionABC API ----

    async def get_items(self, limit: int | None = None) -> List[TResponseInputItem]:
        """Return history trimmed to the last N user turns (optionally limited to most-recent `limit` items)."""
        async with self._lock:
            trimmed = self._trim_to_last_turns(list(self._items))
            return trimmed[-limit:] if (limit is not None and limit >= 0) else trimmed

    async def add_items(self, items: List[TResponseInputItem]) -> None:
        """Append new items, then trim to last N user turns."""
        if not items:
            return
        async with self._lock:
            self._items.extend(items)
            trimmed = self._trim_to_last_turns(list(self._items))
            self._items.clear()
            self._items.extend(trimmed)

    async def pop_item(self) -> TResponseInputItem | None:
        """Remove and return the most recent item (post-trim)."""
        async with self._lock:
            return self._items.pop() if self._items else None

    async def clear_session(self) -> None:
        """Remove all items for this session."""
        async with self._lock:
            self._items.clear()

    # ---- Helpers ----

    def _trim_to_last_turns(self, items: List[TResponseInputItem]) -> List[TResponseInputItem]:
        """
        Keep only the suffix containing the last `max_turns` user messages and everything after
        the earliest of those user messages.

        If there are fewer than `max_turns` user messages (or none), keep all items.
        """
        if not items:
            return items

        count = 0
        start_idx = 0  # default: keep all if we never reach max_turns

        # Walk backward; when we hit the Nth user message, mark its index.
        for i in range(len(items) - 1, -1, -1):
            if _is_user_msg(items[i]):
                count += 1
                if count == self.max_turns:
                    start_idx = i
                    break

        return items[start_idx:]

    # ---- Optional convenience API ----

    async def set_max_turns(self, max_turns: int) -> None:
        async with self._lock:
            self.max_turns = max(1, int(max_turns))
            trimmed = self._trim_to_last_turns(list(self._items))
            self._items.clear()
            self._items.extend(trimmed)

    async def raw_items(self) -> List[TResponseInputItem]:
        """Return the untrimmed in-memory log (for debugging)."""
        async with self._lock:
            return list(self._items)
```

実装したカスタム Session オブジェクトを `max_turns=3` で定義してみましょう。

```python
# Keep only the last 8 turns (user + assistant/tool interactions)
session = TrimmingSession("my_session", max_turns=3)
```

**適切な `max_turns` の選び方は？**

このパラメータの決定には、通常、会話履歴を用いた実験が必要です。1 つのアプローチは、複数の会話にわたるターン総数を抽出し、その分布を分析することです。別の選択肢は、LLM を用いて会話を評価し――各会話にいくつのタスクや課題が含まれるかを特定し、課題あたりに必要な平均ターン数を計算することです。

```python
message = "There is a red light blinking on my laptop."
```
```python
result = await Runner.run(
    support_agent,
    message,
    session=session
)
```
```python
history = await session.get_items()
```
```python
history
```
```text
[{'content': 'There is a red light blinking on my laptop.', 'role': 'user'},
 {'id': 'rs_68be66229c008190aa4b3c5501f397080fdfa41323fb39cb',
  'summary': [],
  'type': 'reasoning',
  'content': []},
 {'id': 'msg_68be662f704c8190969bdf539701a3e90fdfa41323fb39cb',
  'content': [{'annotations': [],
    'text': 'A blinking red light usually indicates a power/battery or hardware fault, but the meaning varies by brand.\n\nWhat is the exact make and model of your laptop?\n\nWhile you check that, please try these quick checks:\n1) Note exactly where the red LED is (charging port, power button, keyboard edge) and the blink pattern (e.g., constant blink, 2 short/1 long).\n2) Plug the charger directly into a known‑good wall outlet (no power strip), ensure the charger tip is fully seated, and look for damage to the cable/port. See if the LED behavior changes.\n3) Leave it on charge for 30 minutes in case the battery is critically low.\n4) Power reset: unplug the charger; if the battery is removable, remove it. Hold the power button for 20–30 seconds. Reconnect power (and battery) and try turning it on.\n5) Tell me the LED location, blink pattern, and what changed after these steps.',
    'type': 'output_text',
    'logprobs': []}],
  'role': 'assistant',
  'status': 'completed',
  'type': 'message'}]
```
```python
# Example flow
await session.add_items([{"role": "user", "content": "I am using a macbook pro and it has some overheating issues too."}])
await session.add_items([{"role": "assistant", "content": "I see. Let's check your firmware version."}])
await session.add_items([{"role": "user", "content": "Firmware v1.0.3; still failing."}])
await session.add_items([{"role": "assistant", "content": "Could you please try a factory reset?"}])
await session.add_items([{"role": "user", "content": "Reset done; error 42 now."}])
await session.add_items([{"role": "assistant", "content": "Leave it on charge for 30 minutes in case the battery is critically low. Is there any other error message?"}])
await session.add_items([{"role": "user", "content": "Yes, I see error 404 now."}])
await session.add_items([{"role": "assistant", "content": "Do you see it on the browser while accessing a website?"}])
# At this point, with max_turns=3, everything *before* the earliest of the last 3 user
# messages is summarized into a synthetic pair, and the last 3 turns remain verbatim.

history = await session.get_items()
# Pass `history` into your agent runner / responses call as the conversation context.
```
```python
len(history)
```
```text
6
```
```python
history
```
```text
[{'role': 'user', 'content': 'Firmware v1.0.3; still failing.'},
 {'role': 'assistant', 'content': 'Could you please try a factory reset?'},
 {'role': 'user', 'content': 'Reset done; error 42 now.'},
 {'role': 'assistant',
  'content': 'Leave it on charge for 30 minutes in case the battery is critically low. Is there any other error message?'},
 {'role': 'user', 'content': 'Yes, I see error 404 now.'},
 {'role': 'assistant',
  'content': 'Do you see it on the browser while accessing a website?'}]
```

以下に、`max_turns=3` の場合にトリミングセッションがどのように動作するかを示します。

![Session におけるコンテキストトリミング](https://developers.openai.com/cookbook/assets/images/trimingSession.jpg)

**「ターン」として数えられるもの**

- **ターン**＝ 1 つの **user** メッセージ **＋それに続くすべて**（assistant の返信、推論、ツール呼び出し、ツール結果）で、**次の user メッセージまで**。

**トリミングが起きるタイミング**

- **書き込み時**：`add_items(...)` が新しいアイテムを追加し、その直後に保存済み履歴をトリミングします。
- **読み取り時**：`get_items(...)` は**トリミング済み**のビューを返します（そのため、書き込みを回避したとしても、読み取りで古いターンが漏れることはありません）。

**何を保持するかの決め方**

1. `role == "user"` を持つ任意のアイテムを **user メッセージ**として扱う（`_is_user_msg` 経由）。
2. 履歴を**後方へ**スキャンし、直近 **N** 個（`max_turns`）の user メッセージのインデックスを収集する。
3. それら N 個の user メッセージのうち**最も古い**インデックスを見つける。
4. **そのインデックスから末尾までをすべて保持**し、それより前をすべて破棄する。

これにより各ターンの境界が完全に保たれます。最も古い保持対象の user メッセージがインデックス `k` にあるなら、`k` の後に来たすべての assistant/ツールのアイテムも保持されます。

**小さな例**

履歴（古い → 新しい）：

```plaintext
0: user("Hi")
1: assistant("Hello!")
2: tool_call("lookup")
3: tool_result("…")
4: user("It didn't work")
5: assistant("Try rebooting")
6: user("Rebooted, now error 42")
7: assistant("On it")
```

`max_turns = 2` の場合、直近 2 つの user メッセージはインデックス **4** と **6** にあります。そのうち最も古いのは **4** → アイテム **4..7** を保持し、**0..3** を破棄します。

**なぜこれがうまくいくのか**

- 常に**完全な**ターンを保持するため、assistant は必要な直近のコンテキスト（ユーザーの最後の依頼と、その間の assistant/ツールのステップの両方）を保てます。
- 古いターンを単なるメッセージ単位ではなく丸ごと破棄することで、コンテキストの肥大化を防ぎます。

**カスタマイズのつまみ**

- 初期化時に `max_turns` を変更する。
- アイテムのスキーマが異なる場合は `_is_user_msg(...)` を調整する。
- **メッセージ数**や**トークン数**で上限を設けたい場合は、`_trim_to_last_turns(...)` を置き換えるか、トークンを測定する 2 回目のパスを追加する。

## コンテキスト要約

履歴が `max_turns` を超えると、直近 N 個の user ターンをそのまま保持し、**それより古いものすべてを 2 つの合成メッセージに要約します**：

- `user`：*「Summarize the conversation we had so far.（これまでの会話を要約して。）」*
- `assistant`：*{生成された要約}*

要約を要求する user からのシャドウプロンプトは、user と assistant の間のチャットの流れを混乱させずに会話の自然な流れを保つために追加されます。生成された要約の最終版が assistant メッセージに注入されます。

**要約プロンプト**

よく練られた要約プロンプトは会話のコンテキストを保持するために不可欠であり、常に具体的なユースケースに合わせて調整すべきです。これは、**カスタマーサポートエージェントが次のエージェントへ案件を引き継ぐ**ようなものだと考えてください。彼らがスムーズに続けるために必要な、簡潔かつ重要な詳細は何でしょうか？ プロンプトは適切なバランスを取るべきです――不要な情報で過負荷にせず、かといって重要なコンテキストが失われるほど希薄でもないように。このバランスを達成するには、詳細度を微調整するための慎重な設計と継続的な実験が必要です。

```python
SUMMARY_PROMPT = """
You are a senior customer-support assistant for tech devices, setup, and software issues.
Compress the earlier conversation into a precise, reusable snapshot for future turns.

Before you write (do this silently):
- Contradiction check: compare user claims with system instructions and tool definitions/logs; note any conflicts or reversals.
- Temporal ordering: sort key events by time; the most recent update wins. If timestamps exist, keep them.
- Hallucination control: if any fact is uncertain/not stated, mark it as UNVERIFIED rather than guessing.

Write a structured, factual summary ≤ 200 words using the sections below (use the exact headings):

• Product & Environment:
  - Device/model, OS/app versions, network/context if mentioned.

• Reported Issue:
  - Single-sentence problem statement (latest state).

• Steps Tried & Results:
  - Chronological bullets (include tool calls + outcomes, errors, codes).

• Identifiers:
  - Ticket #, device serial/model, account/email (only if provided).

• Timeline Milestones:
  - Key events with timestamps or relative order (e.g., 10:32 install → 10:41 error).

• Tool Performance Insights:
  - What tool calls worked/failed and why (if evident).

• Current Status & Blockers:
  - What's resolved vs pending; explicit blockers preventing progress.

• Next Recommended Step:
  - One concrete action (or two alternatives) aligned with policies/tools.

Rules:
- Be concise, no fluff; use short bullets, verbs first.
- Do not invent new facts; quote error strings/codes exactly when available.
- If previous info was superseded, note "Superseded:" and omit details unless critical.
"""
```

**メモリ要約プロンプトを設計するための主要原則**

- **マイルストーン：** 会話における重要なイベントを強調します――例えば、課題が解決したとき、価値ある情報が判明したとき、必要な詳細がすべて集まったとき。
- **ユースケース固有性：** 圧縮プロンプトを具体的なユースケースに合わせて調整します。同じタスクを解決する際、人間ならどのように作業記憶の中で情報を追跡し想起するかを考えてください。
- **矛盾チェック：** 要約がそれ自身、システム指示、ツール定義と矛盾しないことを保証します。これは特に、コンテキスト内での矛盾を起こしやすい推論モデルにとって極めて重要です。
- **タイムスタンプと時間的流れ：** 要約にイベントのタイミングを組み込みます。これはモデルが更新を順序立てて推論するのを助け、タイムライン上で最新のメモリを忘れたり覚えたりする際の混乱を減らします。
- **チャンク化：** 詳細を長い段落ではなくカテゴリやセクションに整理します。構造化されたグループ分けは、LLM が情報の断片間の関係を理解する能力を高めます。
- **ツール性能の洞察：** 複数ターンのツール対応対話から得た教訓を捉えます――例えば、どのツールが特定のクエリに対して効果的に機能したか、その理由。これらの洞察は将来のステップを導くのに有用です。
- **ガイダンスと例：** 明確なガイダンスで要約を方向付けます。可能なら、会話履歴から具体例を抽出して、将来のターンをより根拠づけられたコンテキスト豊かなものにします。
- **幻覚の制御：** 含める内容に正確を期します。要約のわずかな幻覚でさえ将来へ伝播し、不正確さで将来のコンテキストを汚染しうるからです。
- **モデル選択：** ユースケースの要件、要約の長さ、レイテンシとコストのトレードオフに基づいて要約器モデルを選びます。場合によっては、AI エージェント自身と同じモデルを使うことが有利なこともあります。
```python
class LLMSummarizer:
    def __init__(self, client, model="gpt-4o", max_tokens=400, tool_trim_limit=600):
        self.client = client
        self.model = model
        self.max_tokens = max_tokens
        self.tool_trim_limit = tool_trim_limit

    async def summarize(self, messages: List[Item]) -> Tuple[str, str]:
        """
        Create a compact summary from `messages`.

        Returns:
            Tuple[str, str]: The shadow user line to keep dialog natural,
            and the model-generated summary text.
        """
        user_shadow = "Summarize the conversation we had so far."
        TOOL_ROLES = {"tool", "tool_result"}

        def to_snippet(m: Item) -> str | None:
            role = (m.get("role") or "assistant").lower()
            content = (m.get("content") or "").strip()
            if not content:
                return None
            # Trim verbose tool outputs to keep prompt compact    
            if role in TOOL_ROLES and len(content) > self.tool_trim_limit:
                content = content[: self.tool_trim_limit] + " …"
            return f"{role.upper()}: {content}"

        # Build compact, trimmed history
        history_snippets = [s for m in messages if (s := to_snippet(m))]

        prompt_messages = [
            {"role": "system", "content": SUMMARY_PROMPT},
            {"role": "user", "content": "\n".join(history_snippets)},
        ]

        resp = await asyncio.to_thread(
            self.client.responses.create,
            model=self.model,
            input=prompt_messages,
            max_output_tokens=self.max_tokens,
        )

        summary = resp.output_text
        await asyncio.sleep(0)  # yield control
        return user_shadow, summary
```
```python
import asyncio
from collections import deque
from typing import Optional, List, Tuple, Dict, Any

Record = Dict[str, Dict[str, Any]]  # {"msg": {...}, "meta": {...}}

class SummarizingSession:
    """
    Session that keeps only the last N *user turns* verbatim and summarizes the rest.

    - A *turn* starts at a real user message and includes everything until the next real user message.
    - When the number of real user turns exceeds `context_limit`, everything before the earliest
      of the last `keep_last_n_turns` user-turn starts is summarized into a synthetic user→assistant pair.
    - Stores full records (message + metadata). Exposes:
        • get_items():           model-safe messages only (no metadata)
        • get_full_history():    [{"message": msg, "metadata": meta}, ...]
    """

    # Only these keys are ever sent to the model; the rest live in metadata.
    _ALLOWED_MSG_KEYS = {"role", "content", "name"}

    def __init__(
        self,
        keep_last_n_turns: int = 3,
        context_limit: int = 3,
        summarizer: Optional["Summarizer"] = None,
        session_id: Optional[str] = None,
    ):
        assert context_limit >= 1
        assert keep_last_n_turns >= 0
        assert keep_last_n_turns <= context_limit, "keep_last_n_turns should not be greater than context_limit"

        self.keep_last_n_turns = keep_last_n_turns
        self.context_limit = context_limit
        self.summarizer = summarizer
        self.session_id = session_id or "default"

        self._records: deque[Record] = deque()
        self._lock = asyncio.Lock()

    # --------- public API used by your runner ---------
    async def get_items(self, limit: Optional[int] = None) -> List[Dict[str, Any]]:
        """Return model-safe messages only (no metadata)."""
        async with self._lock:
            data = list(self._records)
        msgs = [self._sanitize_for_model(rec["msg"]) for rec in data]
        return msgs[-limit:] if limit else msgs

    async def add_items(self, items: List[Dict[str, Any]]) -> None:
        """Append new items and, if needed, summarize older turns."""
        # 1) Ingest items
        async with self._lock:
            for it in items:
                msg, meta = self._split_msg_and_meta(it)
                self._records.append({"msg": msg, "meta": meta})

            need_summary, boundary = self._summarize_decision_locked()

        # 2) No summarization needed → just normalize flags and exit
        if not need_summary:
            async with self._lock:
                self._normalize_synthetic_flags_locked()
            return

        # 3) Prepare summary prefix (model-safe copy) outside the lock
        async with self._lock:
            snapshot = list(self._records)
            prefix_msgs = [r["msg"] for r in snapshot[:boundary]]

        user_shadow, assistant_summary = await self._summarize(prefix_msgs)

        # 4) Re-check and apply summary atomically
        async with self._lock:
            still_need, new_boundary = self._summarize_decision_locked()
            if not still_need:
                self._normalize_synthetic_flags_locked()
                return

            snapshot = list(self._records)
            suffix = snapshot[new_boundary:]  # keep-last-N turns live here

            # Replace with: synthetic pair + suffix
            self._records.clear()
            self._records.extend([
                {
                    "msg": {"role": "user", "content": user_shadow},
                    "meta": {
                        "synthetic": True,
                        "kind": "history_summary_prompt",
                        "summary_for_turns": f"< all before idx {new_boundary} >",
                    },
                },
                {
                    "msg": {"role": "assistant", "content": assistant_summary},
                    "meta": {
                        "synthetic": True,
                        "kind": "history_summary",
                        "summary_for_turns": f"< all before idx {new_boundary} >",
                    },
                },
            ])
            self._records.extend(suffix)

            # Ensure all real user/assistant messages explicitly have synthetic=False
            self._normalize_synthetic_flags_locked()

    async def pop_item(self) -> Optional[Dict[str, Any]]:
        """Pop the latest message (model-safe), if any."""
        async with self._lock:
            if not self._records:
                return None
            rec = self._records.pop()
            return dict(rec["msg"])

    async def clear_session(self) -> None:
        """Remove all records."""
        async with self._lock:
            self._records.clear()

    def set_max_turns(self, n: int) -> None:
        """
        Back-compat shim for old callers: update `context_limit`
        and clamp `keep_last_n_turns` if needed.
        """
        assert n >= 1
        self.context_limit = n
        if self.keep_last_n_turns > self.context_limit:
            self.keep_last_n_turns = self.context_limit

    # Full history (debugging/analytics/observability)

    async def get_full_history(self, limit: Optional[int] = None) -> List[Dict[str, Any]]:
        """
        Return combined history entries in the shape:
          {"message": {role, content[, name]}, "metadata": {...}}
        This is NOT sent to the model; for logs/UI/debugging only.
        """
        async with self._lock:
            data = list(self._records)
        out = [{"message": dict(rec["msg"]), "metadata": dict(rec["meta"])} for rec in data]
        return out[-limit:] if limit else out

    # Back-compat alias
    async def get_items_with_metadata(self, limit: Optional[int] = None) -> List[Dict[str, Any]]:
        return await self.get_full_history(limit)

    # Internals

    def _split_msg_and_meta(self, it: Dict[str, Any]) -> Tuple[Dict[str, Any], Dict[str, Any]]:
        """
        Split input into (msg, meta):
          - msg keeps only _ALLOWED_MSG_KEYS; if role/content missing, default them.
          - everything else goes under meta (including nested "metadata" if provided).
          - default synthetic=False for real user/assistant unless explicitly set.
        """
        msg = {k: v for k, v in it.items() if k in self._ALLOWED_MSG_KEYS}
        extra = {k: v for k, v in it.items() if k not in self._ALLOWED_MSG_KEYS}
        meta = dict(extra.pop("metadata", {}))
        meta.update(extra)

        msg.setdefault("role", "user")
        msg.setdefault("content", str(it))

        role = msg.get("role")
        if role in ("user", "assistant") and "synthetic" not in meta:
            meta["synthetic"] = False
        return msg, meta

    @staticmethod
    def _sanitize_for_model(msg: Dict[str, Any]) -> Dict[str, Any]:
        """Drop anything not allowed in model calls."""
        return {k: v for k, v in msg.items() if k in SummarizingSession._ALLOWED_MSG_KEYS}

    @staticmethod
    def _is_real_user_turn_start(rec: Record) -> bool:
        """True if record starts a *real* user turn (role=='user' and not synthetic)."""
        return (
            rec["msg"].get("role") == "user"
            and not rec["meta"].get("synthetic", False)
        )

    def _summarize_decision_locked(self) -> Tuple[bool, int]:
        """
        Decide whether to summarize and compute the boundary index.

        Returns:
            (need_summary, boundary_idx)

        If need_summary:
          • boundary_idx is the earliest index among the last `keep_last_n_turns`
            *real* user-turn starts.
          • Everything before boundary_idx becomes the summary prefix.
        """
        user_starts: List[int] = [
            i for i, rec in enumerate(self._records) if self._is_real_user_turn_start(rec)
        ]
        real_turns = len(user_starts)

        # Not over the limit → nothing to do
        if real_turns <= self.context_limit:
            return False, -1

        # Keep zero turns verbatim → summarize everything
        if self.keep_last_n_turns == 0:
            return True, len(self._records)

        # Otherwise, keep the last N turns; summarize everything before the earliest of those
        if len(user_starts) < self.keep_last_n_turns:
            return False, -1  # defensive (shouldn't happen given the earlier check)

        boundary = user_starts[-self.keep_last_n_turns]

        # If there is nothing before boundary, there is nothing to summarize
        if boundary <= 0:
            return False, -1

        return True, boundary

    def _normalize_synthetic_flags_locked(self) -> None:
        """Ensure all real user/assistant records explicitly carry synthetic=False."""
        for rec in self._records:
            role = rec["msg"].get("role")
            if role in ("user", "assistant") and "synthetic" not in rec["meta"]:
                rec["meta"]["synthetic"] = False

    async def _summarize(self, prefix_msgs: List[Dict[str, Any]]) -> Tuple[str, str]:
        """
        Ask the configured summarizer to compress the given prefix.
        Uses model-safe messages only. If no summarizer is configured,
        returns a graceful fallback.
        """
        if not self.summarizer:
            return ("Summarize the conversation we had so far.", "Summary unavailable.")
        clean_prefix = [self._sanitize_for_model(m) for m in prefix_msgs]
        return await self.summarizer.summarize(clean_prefix)
```

**高レベルの考え方**

- **ターン**＝ 1 つの**実際の user** メッセージ **＋それに続くすべて**（assistant の返信、ツール呼び出し/結果など）で、**次の実際の user メッセージまで**。
- 2 つのつまみを設定します：
	- **`context_limit`**：要約する前に raw 履歴に許される**実際の user ターン**の最大数。
		- **`keep_last_n_turns`**：要約を行う際に、直近の**ターン**を何個そのまま保持するか。
		- 不変条件：`keep_last_n_turns <= context_limit`。
- **実際の** user ターン数が `context_limit` を超えると、セッションは：
	1. 直近 `keep_last_n_turns` 個のターン開始のうち**最も古いもの**より**前**のすべてを**要約し**、
		2. 保持領域の先頭に**合成 user→assistant ペア**を注入します：
		- `user`：`"Summarize the conversation we had so far."`（シャドウプロンプト）
				- `assistant`：`{生成された要約}`
		3. 直近 `keep_last_n_turns` 個のターンを**そのまま（verbatim）保持**します。

これにより、直近 `keep_last_n_turns` 個のターンが起きたとおり正確に保持される一方、それより前の内容はすべて 2 つの合成メッセージに圧縮されることが保証されます。

```python
session = SummarizingSession(
    keep_last_n_turns=2,
    context_limit=4,
    summarizer=LLMSummarizer(client)
)
```
```python
# Example flow
await session.add_items([{"role": "user", "content": "Hi, my router won't connect. by the way, I am using Windows 10. I tried troubleshooting via your FAQs but I didn't get anywhere. This is my third tiem calling you. I am based in the US and one of Premium customers."}])
await session.add_items([{"role": "assistant", "content": "Let's check your firmware version."}])
await session.add_items([{"role": "user", "content": "Firmware v1.0.3; still failing."}])
await session.add_items([{"role": "assistant", "content": "Try a factory reset."}])
await session.add_items([{"role": "user", "content": "Reset done; error 42 now."}])
await session.add_items([{"role": "assistant", "content": "Try to install a new firmware."}])
await session.add_items([{"role": "user", "content": "I tried but I got another error now."}])
await session.add_items([{"role": "assistant", "content": "Can you please provide me with the error code?"}])
await session.add_items([{"role": "user", "content": "It says 404 not found when I try to access the page."}])
await session.add_items([{"role": "assistant", "content": "Are you connected to the internet?"}])
# At this point, with context_limit=4, everything *before* the earliest of the last 4 turns
# is summarized into a synthetic pair, and the last 2 turns remain verbatim.
```
```python
history = await session.get_items()
# Pass `history` into your agent runner / responses call as the conversation context.
```
```python
history
```
```text
[{'role': 'user', 'content': 'Summarize the conversation we had so far.'},
 {'role': 'assistant',
  'content': '• Product & Environment:\n  - Router with Firmware v1.0.3, Windows 10, based in the US.\n\n• Reported Issue:\n  - Router fails to connect.\n\n• Steps Tried & Results:\n  - Checked FAQs: No resolution.\n  - Checked firmware version: v1.0.3, problem persists.\n  - Factory reset: Resulted in error 42.\n\n• Identifiers:\n  - Premium customer (no specific identifier provided).\n\n• Timeline Milestones:\n  - Initial troubleshooting via FAQs.\n  - Firmware check (before factory reset).\n  - Factory reset → Error 42.\n\n• Tool Performance Insights:\n  - Firmware version check successful.\n  - Factory reset resulted in new error (42).\n\n• Current Status & Blockers:\n  - Connection issue unresolved; error 42 is the immediate blocker.\n\n• Next Recommended Step:\n  - Install a new firmware update.'},
 {'role': 'user', 'content': 'I tried but I got another error now.'},
 {'role': 'assistant',
  'content': 'Can you please provide me with the error code?'},
 {'role': 'user',
  'content': 'It says 404 not found when I try to access the page.'},
 {'role': 'assistant', 'content': 'Are you connected to the internet?'}]
```
```python
print(history[1]['content'])
```
```text
• Product & Environment:
  - Router with Firmware v1.0.3, Windows 10, based in the US.

• Reported Issue:
  - Router fails to connect.

• Steps Tried & Results:
  - Checked FAQs: No resolution.
  - Checked firmware version: v1.0.3, problem persists.
  - Factory reset: Resulted in error 42.

• Identifiers:
  - Premium customer (no specific identifier provided).

• Timeline Milestones:
  - Initial troubleshooting via FAQs.
  - Firmware check (before factory reset).
  - Factory reset → Error 42.

• Tool Performance Insights:
  - Firmware version check successful.
  - Factory reset resulted in new error (42).

• Current Status & Blockers:
  - Connection issue unresolved; error 42 is the immediate blocker.

• Next Recommended Step:
  - Install a new firmware update.
```

`get_items_with_metadata` メソッドを使うと、デバッグや分析の目的で、メタデータを含むセッションの完全な履歴を取得できます。

```python
full_history = await session.get_items_with_metadata()
```
```python
full_history
```
```text
[{'message': {'role': 'user',
   'content': 'Summarize the conversation we had so far.'},
  'metadata': {'synthetic': True,
   'kind': 'history_summary_prompt',
   'summary_for_turns': '< all before idx 6 >'}},
 {'message': {'role': 'assistant',
   'content': '**Product & Environment:**\n- Device: Router\n- OS: Windows 10\n- Firmware: v1.0.3\n\n**Reported Issue:**\n- Router fails to connect to the internet, now showing error 42.\n\n**Steps Tried & Results:**\n- Checked FAQs: No resolution.\n- Firmware version checked: v1.0.3.\n- Factory reset performed: Resulted in error 42.\n\n**Identifiers:**\n- UNVERIFIED\n\n**Timeline Milestones:**\n- User attempted FAQ troubleshooting.\n- Firmware checked after initial advice.\n- Factory reset led to error 42.\n\n**Tool Performance Insights:**\n- FAQs and basic reset process did not resolve the issue.\n\n**Current Status & Blockers:**\n- Error 42 unresolved; firmware update needed.\n\n**Next Recommended Step:**\n- Install the latest firmware update and check for resolution.'},
  'metadata': {'synthetic': True,
   'kind': 'history_summary',
   'summary_for_turns': '< all before idx 6 >'}},
 {'message': {'role': 'user',
   'content': 'I tried but I got another error now.'},
  'metadata': {'synthetic': False}},
 {'message': {'content': 'I still have a problem with my router.',
   'role': 'user'},
  'metadata': {'synthetic': False}},
 {'message': {'content': [], 'role': 'user'},
  'metadata': {'id': 'rs_68ba192de700819dbed28ad768a9c48205277fe33200f1e3',
   'summary': [],
   'type': 'reasoning',
   'synthetic': False}},
 {'message': {'content': [{'annotations': [],
     'text': 'Sorry you're still stuck. What is the exact error code/message you see now during the firmware update, and does it appear in the router's web UI or elsewhere?\n\nWhile you check that, try these quick, safe steps:\n1) Verify the firmware file exactly matches your router's model and hardware revision (check the label on the router) and region.\n2) Re‑download the firmware from the vendor site and verify its checksum (MD5/SHA256) if provided.\n3) Use a wired Ethernet connection to a LAN port, disable Wi‑Fi on the PC, and try a different browser with extensions disabled.\n4) Ensure you're uploading the correct file type (e.g., .bin/.img), not a ZIP; don't rename the file.\n5) Reboot the router and your PC, then retry the upload; after starting the update, wait at least 10 minutes and don't power off.\n\nNote: "Error 42" meanings vary by brand; once you share the exact current error text and where it appears, I'll give brand‑specific steps (including recovery options if needed).',
     'type': 'output_text',
     'logprobs': []}],
   'role': 'assistant'},
  'metadata': {'id': 'msg_68ba19400060819db38bcb891e9aec7605277fe33200f1e3',
   'status': 'completed',
   'type': 'message',
   'synthetic': False}}]
```
```python
print(history[1]['content'])
```
```text
**Product & Environment:**
- Device: Router
- OS: Windows 10
- Firmware: v1.0.3

**Reported Issue:**
- Router fails to connect to the internet, now showing error 42.

**Steps Tried & Results:**
- Checked FAQs: No resolution.
- Firmware version checked: v1.0.3.
- Factory reset performed: Resulted in error 42.

**Identifiers:**
- UNVERIFIED

**Timeline Milestones:**
- User attempted FAQ troubleshooting.
- Firmware checked after initial advice.
- Factory reset led to error 42.

**Tool Performance Insights:**
- FAQs and basic reset process did not resolve the issue.

**Current Status & Blockers:**
- Error 42 unresolved; firmware update needed.

**Next Recommended Step:**
- Install the latest firmware update and check for resolution.
```

### メモと設計上の選択

- **「新しい」側でターン境界を保持：** **`keep_last_n_turns` 個の user ターン**はそのまま残り、それより古いものはすべて圧縮されます。
- **2 メッセージの要約ブロック：** 下流のツールが検出・表示しやすい（`metadata.synthetic == True`）。
- **Async ＋ ロックの規律：** （潜在的に遅い）要約処理の実行中は**ロックを解放**し、その後に条件を再チェックしてから要約を適用することで、競合的なマージを避けます。
- **冪等な挙動：** 要約中にさらにメッセージが届いても、await 後の再チェックが古い書き換えを防ぎます。

## 評価（Evals）

結局のところ、コンテキストエンジニアリングにおいても**評価（evals）こそすべて**です。問うべき核心は次のとおりです：*モデルが「コンテキストを失っていない」「コンテキストを取り違えていない」ことを、どうやって知るのか？*

メモリに関する完全なクックブックは将来それ単独で成立しうるものですが、ここでは手始めとなる軽量な評価ハーネスのアイデアをいくつか挙げます：

- **ベースラインと差分（Deltas）：** コアとなる評価セットを継続的に実行し、実験の前後を比較してメモリの改善を測定します。
- **LLM-as-Judge：** 注意深く設計された採点プロンプトを持つモデルを用いて、要約の品質を評価します。最も重要な詳細を正しい形式で捉えているかに焦点を当てます。
- **トランスクリプト再生（Transcript Replay）：** 長い会話を再実行し、コンテキストトリミングの有無で次ターンの精度を測定します。指標には、エンティティ/ID の完全一致や、推論品質のルーブリックに基づくスコアリングが含まれます。
- **エラー回帰の追跡：** よくある失敗モード――未回答の質問、落とされた制約、不要/反復的なツール呼び出し――を監視します。
- **トークン圧迫チェック：** トークン上限が保護すべきコンテキストの破棄を強いるケースをフラグ付けします。前後のトークン数をログに記録し、重要な詳細がプルーニングされている時点を検出します。

---
