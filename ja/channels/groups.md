

  設定

  
# グループ

OpenClawは、WhatsApp、Telegram、Discord、Slack、Signal、iMessage、Microsoft Teams、Zaloなどの各プラットフォームでグループチャットを一貫して扱います。

## 初心者向け紹介 (2分)

OpenClawはあなた自身のメッセージングアカウント上で「動作」します。別個のWhatsAppボットユーザーは存在しません。**あなた**がグループに参加している場合、OpenClawはそのグループを認識し、そこで応答できます。デフォルトの動作は以下の通りです：

-   グループは制限されています (`groupPolicy: "allowlist"`)。
-   明示的にメンションゲーティングを無効にしない限り、応答にはメンションが必要です。

要約：許可リストに登録された送信者は、OpenClawをメンションすることで起動できます。

> TL;DR
>
> -   **DMアクセス**は `*.allowFrom` で制御されます。
> -   **グループアクセス**は `*.groupPolicy` + 許可リスト (`*.groups`, `*.groupAllowFrom`) で制御されます。
> -   **応答トリガー**はメンションゲーティング (`requireMention`, `/activation`) で制御されます。

クイックフロー (グループメッセージへの処理)：

```
groupPolicy? disabled -> 破棄
groupPolicy? allowlist -> グループ許可？ いいえ -> 破棄
requireMention? はい -> メンションされた？ いいえ -> コンテキストのみに保存
それ以外 -> 応答
```

![グループメッセージフロー](../images/channels-groups-flow.svg.md) もしあなたが望むなら…

| 目的 | 設定内容 |
| --- | --- |
| すべてのグループを許可するが、@メンション時のみ応答 | `groups: { "*": { requireMention: true } }` |
| すべてのグループ応答を無効化 | `groupPolicy: "disabled"` |
| 特定のグループのみ許可 | `groups: { "<グループID>": { ... } }` (`"*"` キーなし) |
| グループ内であなたのみが起動可能 | `groupPolicy: "allowlist"`, `groupAllowFrom: ["+1555..."]` |

## セッションキー

-   グループセッションは `agent:::group:` セッションキーを使用します (ルーム/チャネルは `agent:::channel:` を使用)。
-   TelegramフォーラムトピックはグループIDに `:topic:` を追加するため、各トピックが独自のセッションを持ちます。
-   ダイレクトチャットはメインセッション (または設定されている場合は送信者ごとのセッション) を使用します。
-   グループセッションではハートビートはスキップされます。

## パターン: 個人DM + 公開グループ (単一エージェント)

可能です — あなたの「個人」トラフィックが**DM**であり、「公開」トラフィックが**グループ**である場合、これはうまく機能します。理由：単一エージェントモードでは、DMは通常**メイン**セッションキー (`agent:main:main`) に着信し、グループは常に**非メイン**セッションキー (`agent:main::group:`) を使用します。`mode: "non-main"` でサンドボックス化を有効にすると、それらのグループセッションはDocker内で実行され、あなたのメインDMセッションはホスト上に残ります。これにより、1つのエージェント「脳」(共有ワークスペース + メモリ) を持ちながら、2つの実行姿勢が得られます：

-   **DM**: フルツール (ホスト)
-   **グループ**: サンドボックス + 制限付きツール (Docker)

> もし本当に分離されたワークスペース/ペルソナが必要な場合 (「個人」と「公開」が決して混ざらない)、2番目のエージェント + バインディングを使用してください。 [マルチエージェントルーティング](../concepts/multi-agent.md) を参照。

例 (DMはホスト上、グループはサンドボックス化 + メッセージング専用ツール)：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main", // グループ/チャネルは非メイン -> サンドボックス化
        scope: "session", // 最強の分離 (グループ/チャネルごとに1コンテナ)
        workspaceAccess: "none",
      },
    },
  },
  tools: {
    sandbox: {
      tools: {
        // allowが空でない場合、それ以外はすべてブロックされます (denyは依然として優先)。
        allow: ["group:messaging", "group:sessions"],
        deny: ["group:runtime", "group:fs", "group:ui", "nodes", "cron", "gateway"],
      },
    },
  },
}
```

「グループはフォルダXのみ閲覧可能」ではなく「ホストアクセスなし」を望みますか？ `workspaceAccess: "none"` を維持し、許可リストされたパスのみをサンドボックスにマウントします：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
        docker: {
          binds: [
            // hostPath:containerPath:mode
            "/home/user/FriendsShared:/data:ro",
          ],
        },
      },
    },
  },
}
```

関連項目：

-   設定キーとデフォルト: [ゲートウェイ設定](../gateway/configuration.md#agentsdefaultssandbox)
-   ツールがブロックされる理由のデバッグ: [サンドボックス vs ツールポリシー vs 昇格](../gateway/sandbox-vs-tool-policy-vs-elevated.md)
-   バインドマウントの詳細: [サンドボックス化](../gateway/sandboxing.md#custom-bind-mounts)

## 表示ラベル

-   UIラベルは利用可能な場合 `displayName` を使用し、`:` としてフォーマットされます。
-   `#room` はルーム/チャネル用に予約されています；グループチャットは `g-` を使用します (小文字、スペース -> `-`、`#@+._-` は保持)。

## グループポリシー

チャネルごとにグループ/ルームメッセージの処理方法を制御します：

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "disabled", // "open" | "disabled" | "allowlist"
      groupAllowFrom: ["+15551234567"],
    },
    telegram: {
      groupPolicy: "disabled",
      groupAllowFrom: ["123456789"], // 数値のTelegramユーザーID (ウィザードで@usernameを解決可能)
    },
    signal: {
      groupPolicy: "disabled",
      groupAllowFrom: ["+15551234567"],
    },
    imessage: {
      groupPolicy: "disabled",
      groupAllowFrom: ["chat_id:123"],
    },
    msteams: {
      groupPolicy: "disabled",
      groupAllowFrom: ["user@org.com"],
    },
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        GUILD_ID: { channels: { help: { allow: true } } },
      },
    },
    slack: {
      groupPolicy: "allowlist",
      channels: { "#general": { allow: true } },
    },
    matrix: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["@owner:example.org"],
      groups: {
        "!roomId:example.org": { allow: true },
        "#alias:example.org": { allow: true },
      },
    },
  },
}
```

| ポリシー | 動作 |
| --- | --- |
| `"open"` | グループは許可リストをバイパスします；メンションゲーティングは依然として適用されます。 |
| `"disabled"` | すべてのグループメッセージを完全にブロックします。 |
| `"allowlist"` | 設定された許可リストに一致するグループ/ルームのみを許可します。 |

注意点：

-   `groupPolicy` はメンションゲーティング (これは@メンションを必要とします) とは別です。
-   WhatsApp/Telegram/Signal/iMessage/Microsoft Teams/Zalo: `groupAllowFrom` を使用 (フォールバック: 明示的な `allowFrom`)。
-   DMペアリング承認 (`*-allowFrom` ストアエントリ) はDMアクセスのみに適用されます；グループ送信者認可はグループ許可リストに明示的に残ります。
-   Discord: 許可リストは `channels.discord.guilds..channels` を使用します。
-   Slack: 許可リストは `channels.slack.channels` を使用します。
-   Matrix: 許可リストは `channels.matrix.groups` (ルームID、エイリアス、または名前) を使用します。送信者を制限するには `channels.matrix.groupAllowFrom` を使用します；ルームごとの `users` 許可リストもサポートされています。
-   グループDMは別々に制御されます (`channels.discord.dm.*`, `channels.slack.dm.*`)。
-   Telegram許可リストはユーザーID (`"123456789"`, `"telegram:123456789"`, `"tg:123456789"`) またはユーザー名 (`"@alice"` または `"alice"`) に一致できます；プレフィックスは大文字小文字を区別しません。
-   デフォルトは `groupPolicy: "allowlist"` です；グループ許可リストが空の場合、グループメッセージはブロックされます。
-   ランタイム安全性：プロバイダーブロックが完全に欠落している場合 (`channels.` 不在)、グループポリシーは `channels.defaults.groupPolicy` を継承する代わりに、フェイルクローズモード (通常 `allowlist`) にフォールバックします。

クイックメンタルモデル (グループメッセージの評価順序)：

1.  `groupPolicy` (open/disabled/allowlist)
2.  グループ許可リスト (`*.groups`, `*.groupAllowFrom`, チャネル固有の許可リスト)
3.  メンションゲーティング (`requireMention`, `/activation`)

## メンションゲーティング (デフォルト)

グループメッセージは、グループごとに上書きされない限りメンションを必要とします。デフォルトは `*.groups."*"` の下のサブシステムごとに存在します。ボットメッセージへの返信は、暗黙のメンションとしてカウントされます (チャネルが返信メタデータをサポートしている場合)。これはTelegram、WhatsApp、Slack、Discord、Microsoft Teamsに適用されます。

```json
{
  channels: {
    whatsapp: {
      groups: {
        "*": { requireMention: true },
        "123@g.us": { requireMention: false },
      },
    },
    telegram: {
      groups: {
        "*": { requireMention: true },
        "123456789": { requireMention: false },
      },
    },
    imessage: {
      groups: {
        "*": { requireMention: true },
        "123": { requireMention: false },
      },
    },
  },
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw", "\\+15555550123"],
          historyLimit: 50,
        },
      },
    ],
  },
}
```

注意点：

-   `mentionPatterns` は大文字小文字を区別しない正規表現です。
-   明示的なメンションを提供するプラットフォームは依然として通過します；パターンはフォールバックです。
-   エージェントごとの上書き： `agents.list[].groupChat.mentionPatterns` (複数のエージェントがグループを共有する場合に便利)。
-   メンションゲーティングは、メンション検出が可能な場合 (ネイティブメンションまたは `mentionPatterns` が設定されている場合) のみ適用されます。
-   Discordのデフォルトは `channels.discord.guilds."*"` に存在します (ギルド/チャネルごとに上書き可能)。
-   グループ履歴コンテキストはチャネル間で一様にラップされ、**保留中のみ** (メンションゲーティングによりスキップされたメッセージ) です；グローバルデフォルトには `messages.groupChat.historyLimit` を、上書きには `channels..historyLimit` (または `channels..accounts.*.historyLimit`) を使用します。無効化するには `0` を設定します。

## グループ/チャネルツール制限 (オプション)

一部のチャネル設定は、**特定のグループ/ルーム/チャネル内**で利用可能なツールを制限することをサポートしています。

-   `tools`: グループ全体のツールを許可/拒否します。
-   `toolsBySender`: グループ内での送信者ごとの上書き。明示的なキープレフィックスを使用します： `id:`, `e164:`, `username:`, `name:`, および `"*"` ワイルドカード。レガシーなプレフィックスなしキーも `id:` としてのみマッチングされ、引き続き受け入れられます。

解決順序 (最も具体的なものが優先)：

1.  グループ/チャネル `toolsBySender` マッチ
2.  グループ/チャネル `tools`
3.  デフォルト (`"*"`) `toolsBySender` マッチ
4.  デフォルト (`"*"`) `tools`

例 (Telegram)：

```json
{
  channels: {
    telegram: {
      groups: {
        "*": { tools: { deny: ["exec"] } },
        "-1001234567890": {
          tools: { deny: ["exec", "read", "write"] },
          toolsBySender: {
            "id:123456789": { alsoAllow: ["exec"] },
          },
        },
      },
    },
  },
}
```

注意点：

-   グループ/チャネルツール制限は、グローバル/エージェントツールポリシーに加えて適用されます (拒否は依然として優先)。
-   一部のチャネルは、ルーム/チャネルに対して異なるネストを使用します (例：Discord `guilds.*.channels.*`, Slack `channels.*`, MS Teams `teams.*.channels.*`)。

## グループ許可リスト

`channels.whatsapp.groups`、`channels.telegram.groups`、または `channels.imessage.groups` が設定されている場合、キーはグループ許可リストとして機能します。`"*"` を使用してすべてのグループを許可しつつ、デフォルトのメンション動作を設定できます。一般的な意図 (コピー/ペースト用)：

1.  すべてのグループ応答を無効化

```json
{
  channels: { whatsapp: { groupPolicy: "disabled" } },
}
```

2.  特定のグループのみ許可 (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groups: {
        "123@g.us": { requireMention: true },
        "456@g.us": { requireMention: false },
      },
    },
  },
}
```

3.  すべてのグループを許可するがメンションを要求 (明示的)

```json
{
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

4.  グループ内で所有者のみが起動可能 (WhatsApp)

```json
{
  channels: {
    whatsapp: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["+15551234567"],
      groups: { "*": { requireMention: true } },
    },
  },
}
```

## アクティベーション (所有者のみ)

グループ所有者はグループごとのアクティベーションを切り替えられます：

-   `/activation mention`
-   `/activation always`

所有者は `channels.whatsapp.allowFrom` (または未設定の場合はボット自身のE.164) によって決定されます。コマンドを単独のメッセージとして送信します。他のプラットフォームは現在 `/activation` を無視します。

## コンテキストフィールド

グループ受信ペイロードは以下を設定します：

-   `ChatType=group`
-   `GroupSubject` (既知の場合)
-   `GroupMembers` (既知の場合)
-   `WasMentioned` (メンションゲーティング結果)
-   Telegramフォーラムトピックは `MessageThreadId` と `IsForum` も含みます。

エージェントシステムプロンプトには、新しいグループセッションの最初のターンにグループ紹介が含まれます。モデルに人間のように応答し、Markdownテーブルを避け、リテラルな `\n` シーケンスを入力しないよう思い出させます。

## iMessage固有の詳細

-   ルーティングや許可リスト作成時は `chat_id:` を優先します。
-   チャット一覧： `imsg chats --limit 20`。
-   グループ返信は常に同じ `chat_id` に戻ります。

## WhatsApp固有の詳細

WhatsAppのみの動作 (履歴注入、メンションハンドリングの詳細) については [グループメッセージ](./group-messages.md) を参照してください。

[グループメッセージ](./group-messages.md)[ブロードキャストグループ](./broadcast-groups.md)

---