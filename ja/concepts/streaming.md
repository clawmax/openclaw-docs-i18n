

  メッセージと配信

  
# ストリーミングとチャンキング

OpenClawには2つの独立したストリーミングレイヤーがあります:

-   **ブロックストリーミング (チャネル):** アシスタントが書き込む際に完了した**ブロック**を出力します。これらは通常のチャネルメッセージです（トークンデルタではありません）。
-   **プレビューストリーミング (Telegram/Discord/Slack):** 生成中に一時的な**プレビューメッセージ**を更新します。

現在、チャネルメッセージへの**真のトークンデルタストリーミング**はありません。プレビューストリーミングはメッセージベースです（送信 + 編集/追加）。

## ブロックストリーミング (チャネルメッセージ)

ブロックストリーミングは、アシスタントの出力が利用可能になるにつれて、粗いチャンクで送信します。

```
Model output
  └─ text_delta/events
       ├─ (blockStreamingBreak=text_end)
       │    └─ chunker emits blocks as buffer grows
       └─ (blockStreamingBreak=message_end)
            └─ chunker flushes at message_end
                   └─ channel send (block replies)
```

凡例:

-   `text_delta/events`: モデルストリームイベント (非ストリーミングモデルではまばらな場合があります)。
-   `chunker`: 最小/最大境界 + ブレーク設定を適用する `EmbeddedBlockChunker`。
-   `channel send`: 実際の送信メッセージ (ブロック返信)。

**制御:**

-   `agents.defaults.blockStreamingDefault`: `"on"`/`"off"` (デフォルトはoff)。
-   チャネルオーバーライド: `*.blockStreaming` (およびアカウントごとのバリエーション) でチャネルごとに `"on"`/`"off"` を強制。
-   `agents.defaults.blockStreamingBreak`: `"text_end"` または `"message_end"`。
-   `agents.defaults.blockStreamingChunk`: `{ minChars, maxChars, breakPreference? }`。
-   `agents.defaults.blockStreamingCoalesce`: `{ minChars?, maxChars?, idleMs? }` (送信前にストリームされたブロックをマージ)。
-   チャネルハードキャップ: `*.textChunkLimit` (例: `channels.whatsapp.textChunkLimit`)。
-   チャネルチャンクモード: `*.chunkMode` (`length` デフォルト, `newline` は長さチャンキングの前に空白行 (段落境界) で分割)。
-   Discordソフトキャップ: `channels.discord.maxLinesPerMessage` (デフォルト17) は、UIのクリッピングを避けるために長い返信を分割します。

**境界セマンティクス:**

-   `text_end`: チャンカーが出力するたびにブロックをストリーム; 各 `text_end` でフラッシュ。
-   `message_end`: アシスタントメッセージが終了するまで待機し、バッファリングされた出力をフラッシュ。

`message_end` も、バッファリングされたテキストが `maxChars` を超える場合、チャンカーを使用するため、最後に複数のチャンクを出力する可能性があります。

## チャンキングアルゴリズム (下限/上限)

ブロックチャンキングは `EmbeddedBlockChunker` によって実装されます:

-   **下限:** バッファ >= `minChars` になるまで出力しない (強制されない限り)。
-   **上限:** `maxChars` の前で分割を優先; 強制された場合は `maxChars` で分割。
-   **ブレーク設定:** `paragraph` → `newline` → `sentence` → `whitespace` → ハードブレーク。
-   **コードフェンス:** フェンス内では決して分割しない; `maxChars` で強制された場合、Markdownを有効に保つためにフェンスを閉じて再開。

`maxChars` はチャネルの `textChunkLimit` にクランプされるため、チャネルごとのキャップを超えることはできません。

## 結合 (ストリームされたブロックのマージ)

ブロックストリーミングが有効な場合、OpenClawは送信前に**連続するブロックチャンクをマージ**できます。これにより、進行性のある出力を提供しながら「単一行スパム」を減らします。

-   結合はフラッシュ前に**アイドルギャップ** (`idleMs`) を待機します。
-   バッファは `maxChars` によってキャップされ、それを超えるとフラッシュします。
-   `minChars` は、十分なテキストが蓄積されるまで小さな断片が送信されるのを防ぎます (最終フラッシュは常に残りのテキストを送信)。
-   結合文字は `blockStreamingChunk.breakPreference` から導出されます (`paragraph` → `\n\n`, `newline` → `\n`, `sentence` → スペース)。
-   チャネルオーバーライドは `*.blockStreamingCoalesce` で利用可能です (アカウントごとの設定を含む)。
-   デフォルトの結合 `minChars` は、オーバーライドされない限り、Signal/Slack/Discordでは1500に引き上げられます。

## ブロック間の人間らしい間隔

ブロックストリーミングが有効な場合、ブロック返信 (最初のブロックの後) の間に**ランダム化された一時停止**を追加できます。これにより、複数のバブル応答がより自然に感じられます。

-   設定: `agents.defaults.humanDelay` (エージェントごとに `agents.list[].humanDelay` でオーバーライド可能)。
-   モード: `off` (デフォルト), `natural` (800–2500ms), `custom` (`minMs`/`maxMs`)。
-   **ブロック返信**にのみ適用され、最終返信やツール要約には適用されません。

## 「チャンクをストリームするか、すべてをストリームするか」

これは以下に対応します:

-   **チャンクをストリーム:** `blockStreamingDefault: "on"` + `blockStreamingBreak: "text_end"` (進行に応じて出力)。非Telegramチャネルも `*.blockStreaming: true` が必要。
-   **最後にすべてをストリーム:** `blockStreamingBreak: "message_end"` (一度フラッシュ、非常に長い場合は複数チャンクになる可能性あり)。
-   **ブロックストリーミングなし:** `blockStreamingDefault: "off"` (最終返信のみ)。

**チャネル注意:** ブロックストリーミングは、`*.blockStreaming` が明示的に `true` に設定されない限り**オフです**。チャネルは、ブロック返信なしでライブプレビュー (`channels..streaming`) をストリームできます。設定場所の注意: `blockStreaming*` のデフォルトはルート設定ではなく `agents.defaults` にあります。

## プレビューストリーミングモード

正規キー: `channels..streaming` モード:

-   `off`: プレビューストリーミングを無効化。
-   `partial`: 最新のテキストに置き換えられる単一のプレビュー。
-   `block`: チャンク化/追加されたステップでプレビューを更新。
-   `progress`: 生成中の進捗/ステータスプレビュー、完了時に最終回答。

### チャネルマッピング

| チャネル | `off` | `partial` | `block` | `progress` |
| --- | --- | --- | --- | --- |
| Telegram | ✅ | ✅ | ✅ | `partial` にマッピング |
| Discord | ✅ | ✅ | ✅ | `partial` にマッピング |
| Slack | ✅ | ✅ | ✅ | ✅ |

Slackのみ:

-   `channels.slack.nativeStreaming` は、`streaming=partial` のときにSlackネイティブストリーミングAPI呼び出しを切り替えます (デフォルト: `true`)。

レガシーキーの移行:

-   Telegram: `streamMode` + ブール値 `streaming` は自動的に `streaming` 列挙型に移行。
-   Discord: `streamMode` + ブール値 `streaming` は自動的に `streaming` 列挙型に移行。
-   Slack: `streamMode` は自動的に `streaming` 列挙型に移行; ブール値 `streaming` は自動的に `nativeStreaming` に移行。

### ランタイム動作

Telegram:

-   利用可能な場合はDMでBot API `sendMessageDraft` を使用し、グループ/トピックプレビュー更新には `sendMessage` + `editMessageText` を使用。
-   プレビューストリーミングは、Telegramブロックストリーミングが明示的に有効な場合はスキップされます (二重ストリーミングを避けるため)。
-   `/reasoning stream` は推論をプレビューに書き込むことができます。

Discord:

-   送信 + 編集プレビューメッセージを使用。
-   `block` モードはドラフトチャンキング (`draftChunk`) を使用。
-   プレビューストリーミングは、Discordブロックストリーミングが明示的に有効な場合はスキップされます。

Slack:

-   `partial` は、利用可能な場合にSlackネイティブストリーミング (`chat.startStream`/`append`/`stop`) を使用できます。
-   `block` は追加スタイルのドラフトプレビューを使用。
-   `progress` はステータスプレビューテキストを使用し、その後最終回答。

[メッセージ](./messages.md)[再試行ポリシー](./retry.md)