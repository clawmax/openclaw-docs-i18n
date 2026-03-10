title: "OpenClaw AI エージェントのための Discord のセットアップと設定方法"
description: "OpenClaw AI エージェントを Discord に接続する方法を学びましょう。ボット作成、トークン設定、サーバーとのペアリング、ギルドチャンネルの設定に関するステップバイステップガイドです。"
keywords: ["discord ボット セットアップ", "openclaw discord", "ai エージェント discord", "discord ゲートウェイ", "ボット トークン 設定", "discord ギルド チャンネル", "discord ペアリング", "discord フォーラム チャンネル"]
---

  メッセージングプラットフォーム

  
# Discord

ステータス: 公式 Discord ゲートウェイ経由での DM およびギルドチャンネルに対応済み。

## クイックセットアップ

新しいアプリケーションとボットを作成し、ボットをサーバーに追加し、OpenClaw とペアリングする必要があります。ボットは自身のプライベートサーバーに追加することをお勧めします。まだお持ちでない場合は、[まずサーバーを作成してください](https://support.discord.com/hc/en-us/articles/204849977-How-do-I-create-a-server) (**自分で作成 > 自分と友達のため** を選択)。

### ステップ 1: Discord アプリケーションとボットを作成する

[Discord Developer Portal](https://discord.com/developers/applications) にアクセスし、**New Application** をクリックします。「OpenClaw」のような名前を付けます。サイドバーの **Bot** をクリックします。**Username** を OpenClaw エージェントの呼び名に設定します。

### ステップ 2: 特権インテントを有効にする

**Bot** ページで、下にスクロールして **Privileged Gateway Intents** を有効にします:

-   **Message Content Intent** (必須)
-   **Server Members Intent** (推奨; ロール許可リストと名前からIDへのマッチングに必要)
-   **Presence Intent** (オプション; プレゼンス更新が必要な場合のみ)

### ステップ 3: ボットトークンをコピーする

**Bot** ページで上にスクロールし、**Reset Token** をクリックします。

> **ℹ️** 名前とは異なり、これは最初のトークンを生成します — 何も「リセット」されません。

トークンをコピーしてどこかに保存します。これが **Bot Token** で、すぐに必要になります。

### ステップ 4: 招待 URL を生成し、ボットをサーバーに追加する

サイドバーの **OAuth2** をクリックします。サーバーにボットを追加するための適切な権限を持つ招待 URL を生成します。**OAuth2 URL Generator** まで下にスクロールし、以下を有効にします:

-   `bot`
-   `applications.commands`

**Bot Permissions** セクションが下に表示されます。以下を有効にします:

-   View Channels
-   Send Messages
-   Read Message History
-   Embed Links
-   Attach Files
-   Add Reactions (オプション)

下部に生成された URL をコピーし、ブラウザに貼り付けてサーバーを選択し、**Continue** をクリックして接続します。これで Discord サーバーにボットが表示されるはずです。

### ステップ 5: 開発者モードを有効にして ID を収集する

Discord アプリに戻り、内部 ID をコピーできるように開発者モードを有効にする必要があります。

1.  **User Settings** (アバター横の歯車アイコン) → **Advanced** → **Developer Mode** をオンに切り替えます
2.  サイドバーの**サーバーアイコン**を右クリック → **Copy Server ID**
3.  自身の**アバター**を右クリック → **Copy User ID**

**Server ID** と **User ID** を Bot Token と一緒に保存します — 次のステップでこれら3つを OpenClaw に送信します。

### ステップ 6: サーバーメンバーからの DM を許可する

ペアリングを機能させるには、Discord がボットからあなたへの DM を許可する必要があります。**サーバーアイコン**を右クリック → **Privacy Settings** → **Direct Messages** をオンに切り替えます。これにより、サーバーメンバー（ボットを含む）があなたに DM を送信できるようになります。OpenClaw で Discord DM を使用したい場合は、これを有効にしたままにします。ギルドチャンネルのみを使用する予定の場合は、ペアリング後に DM を無効にできます。

### ステップ 7: ステップ 0: ボットトークンを安全に設定する (チャットで送信しないでください)

Discord ボットトークンは秘密です (パスワードのようなもの)。エージェントにメッセージを送信する前に、OpenClaw を実行しているマシンに設定します。

```bash
openclaw config set channels.discord.token '"YOUR_BOT_TOKEN"' --json
openclaw config set channels.discord.enabled true --json
openclaw gateway
```

OpenClaw がすでにバックグラウンドサービスとして実行されている場合は、代わりに `openclaw gateway restart` を使用します。

### ステップ 8: OpenClaw を設定してペアリングする

既存のチャンネル (例: Telegram) で OpenClaw エージェントとチャットし、次のように伝えます。Discord が最初のチャンネルの場合は、代わりに CLI / config タブを使用してください。

> 「Discord ボットトークンはすでに config に設定しました。User ID `<user_id>` と Server ID `<server_id>` で Discord セットアップを完了してください。」

```json
{
  channels: {
    discord: {
      enabled: true,
      token: "YOUR_BOT_TOKEN",
    },
  },
}
```

### ステップ 9: 最初の DM ペアリングを承認する

ゲートウェイが実行されるまで待ち、Discord でボットに DM を送ります。ボットはペアリングコードで応答します。

既存のチャンネルでペアリングコードをエージェントに送信します:

> 「この Discord ペアリングコードを承認してください: ``」

```bash
openclaw pairing list discord
openclaw pairing approve discord <CODE>
```

ペアリングコードは1時間で期限切れになります。これで Discord の DM 経由でエージェントとチャットできるようになります。

 

> **ℹ️** トークン解決はアカウントを認識します。Config のトークン値が環境変数のフォールバックよりも優先されます。`DISCORD_BOT_TOKEN` はデフォルトアカウントに対してのみ使用されます。

## 推奨: ギルドワークスペースを設定する

DM が機能したら、Discord サーバーを完全なワークスペースとして設定できます。各チャンネルが独自のコンテキストを持つ独自のエージェントセッションを取得します。これは、あなたとボットだけのプライベートサーバーに推奨されます。

### ステップ 1: ギルド許可リストにサーバーを追加する

これにより、エージェントは DM だけでなく、サーバー上の任意のチャンネルで応答できるようになります。

> 「Discord Server ID `<server_id>` をギルド許可リストに追加してください」

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: true,
          users: ["YOUR_USER_ID"],
        },
      },
    },
  },
}
```

### ステップ 2: @メンションなしでの応答を許可する

デフォルトでは、エージェントはギルドチャンネルで @メンションされた場合にのみ応答します。プライベートサーバーの場合、おそらくすべてのメッセージに応答してほしいでしょう。

> 「このサーバーでエージェントが @メンションなしで応答することを許可してください」

```json
{
  channels: {
    discord: {
      guilds: {
        YOUR_SERVER_ID: {
          requireMention: false,
        },
      },
    },
  },
}
```

### ステップ 3: ギルドチャンネルでのメモリを計画する

デフォルトでは、長期メモリ (MEMORY.md) は DM セッションでのみロードされます。ギルドチャンネルは MEMORY.md を自動ロードしません。

> 「Discord チャンネルで質問するとき、MEMORY.md からの長期的なコンテキストが必要な場合は、memory_search または memory_get を使用してください。」

すべてのチャンネルで共有コンテキストが必要な場合は、安定した指示を `AGENTS.md` または `USER.md` に配置します (これらはすべてのセッションに注入されます)。長期的なメモは `MEMORY.md` に保持し、必要に応じてメモリツールでアクセスします。

 次に、Discord サーバーにいくつかのチャンネルを作成し、チャットを開始します。エージェントはチャンネル名を確認でき、各チャンネルは独自の分離されたセッションを取得します — つまり、ワークフローに合わせて `#coding`、`#home`、`#research` などを設定できます。

## ランタイムモデル

-   ゲートウェイが Discord 接続を所有します。
-   返信ルーティングは決定的です: Discord からの受信は Discord に返信します。
-   デフォルト (`session.dmScope=main`) では、ダイレクトチャットはエージェントのメインセッション (`agent:main:main`) を共有します。
-   ギルドチャンネルは分離されたセッションキーです (`agent::discord:channel:`)。
-   グループ DM はデフォルトで無視されます (`channels.discord.dm.groupEnabled=false`)。
-   ネイティブスラッシュコマンドは分離されたコマンドセッションで実行されます (`agent::discord:slash:`)。同時に、`CommandTargetSessionKey` をルーティングされた会話セッションに引き継ぎます。

## フォーラムチャンネル

Discord のフォーラムおよびメディアチャンネルはスレッド投稿のみを受け付けます。OpenClaw はそれらを作成する2つの方法をサポートしています:

-   フォーラム親 (`channel:`) にメッセージを送信してスレッドを自動作成します。スレッドタイトルはメッセージの最初の空でない行を使用します。
-   `openclaw message thread create` を使用して直接スレッドを作成します。フォーラムチャンネルの場合は `--message-id` を渡さないでください。

例: スレッドを作成するためにフォーラム親に送信する

```bash
openclaw message send --channel discord --target channel:<forumId> \
  --message "Topic title\nBody of the post"
```

例: 明示的にフォーラムスレッドを作成する

```bash
openclaw message thread create --channel discord --target channel:<forumId> \
  --thread-name "Topic title" --message "Body of the post"
```

フォーラム親は Discord コンポーネントを受け付けません。コンポーネントが必要な場合は、スレッド自体 (`channel:`) に送信してください。

## インタラクティブコンポーネント

OpenClaw はエージェントメッセージ用に Discord コンポーネント v2 コンテナをサポートしています。`components` ペイロードを含むメッセージツールを使用します。インタラクション結果は通常の受信メッセージとしてエージェントにルーティングされ、既存の Discord `replyToMode` 設定に従います。サポートされるブロック:

-   `text`, `section`, `separator`, `actions`, `media-gallery`, `file`
-   アクション行は最大5つのボタンまたは単一の選択メニューを許可します
-   選択タイプ: `string`, `user`, `role`, `mentionable`, `channel`

デフォルトでは、コンポーネントは1回使用です。`components.reusable=true` を設定すると、ボタン、選択、フォームが期限切れになるまで複数回使用できるようになります。ボタンをクリックできるユーザーを制限するには、そのボタンに `allowedUsers` を設定します (Discord ユーザー ID、タグ、または `*`)。設定すると、一致しないユーザーは一時的な拒否メッセージを受け取ります。`/model` および `/models` スラッシュコマンドは、プロバイダーとモデルのドロップダウン、および Submit ステップを含むインタラクティブなモデルピッカーを開きます。ピッカーの返信は一時的で、呼び出しユーザーのみが使用できます。ファイル添付:

-   `file` ブロックは添付ファイル参照 (`attachment://`) を指す必要があります
-   `media`/`path`/`filePath` 経由で添付ファイルを提供します (単一ファイル); 複数ファイルの場合は `media-gallery` を使用します
-   アップロード名を添付ファイル参照と一致させる必要がある場合は、`filename` を使用して上書きします

モーダルフォーム:

-   最大5つのフィールドを持つ `components.modal` を追加します
-   フィールドタイプ: `text`, `checkbox`, `radio`, `select`, `role-select`, `user-select`
-   OpenClaw は自動的にトリガーボタンを追加します

例:

```json
{
  channel: "discord",
  action: "send",
  to: "channel:123456789012345678",
  message: "Optional fallback text",
  components: {
    reusable: true,
    text: "Choose a path",
    blocks: [
      {
        type: "actions",
        buttons: [
          {
            label: "Approve",
            style: "success",
            allowedUsers: ["123456789012345678"],
          },
          { label: "Decline", style: "danger" },
        ],
      },
      {
        type: "actions",
        select: {
          type: "string",
          placeholder: "Pick an option",
          options: [
            { label: "Option A", value: "a" },
            { label: "Option B", value: "b" },
          ],
        },
      },
    ],
    modal: {
      title: "Details",
      triggerLabel: "Open form",
      fields: [
        { type: "text", label: "Requester" },
        {
          type: "select",
          label: "Priority",
          options: [
            { label: "Low", value: "low" },
            { label: "High", value: "high" },
          ],
        },
      ],
    },
  },
}
```

## アクセス制御とルーティング

`channels.discord.dmPolicy` は DM アクセスを制御します (レガシー: `channels.discord.dm.policy`):

-   `pairing` (デフォルト)
-   `allowlist`
-   `open` (`channels.discord.allowFrom` に `"*"` を含める必要があります; レガシー: `channels.discord.dm.allowFrom`)
-   `disabled`

DM ポリシーが open でない場合、未知のユーザーはブロックされます (または `pairing` モードではペアリングを促されます)。マルチアカウントの優先順位:

-   `channels.discord.accounts.default.allowFrom` は `default` アカウントにのみ適用されます。
-   名前付きアカウントは、自身の `allowFrom` が設定されていない場合、`channels.discord.allowFrom` を継承します。
-   名前付きアカウントは `channels.discord.accounts.default.allowFrom` を継承しません。

配信用の DM ターゲット形式:

-   `user:`
-   `<@id>` メンション

明示的なユーザー/チャンネルターゲット種別が提供されない限り、単なる数値 ID は曖昧であり拒否されます。

```json
{
  channels: {
    discord: {
      groupPolicy: "allowlist",
      guilds: {
        "123456789012345678": {
          requireMention: true,
          ignoreOtherMentions: true,
          users: ["987654321098765432"],
          roles: ["123456789012345678"],
          channels: {
            general: { allow: true },
            help: { allow: true, requireMention: true },
          },
        },
      },
    },
  },
}
```

ギルドメッセージはデフォルトでメンションゲートされています。メンション検出には以下が含まれます:

-   明示的なボットメンション
-   設定されたメンションパターン (`agents.list[].groupChat.mentionPatterns`, フォールバック `messages.groupChat.mentionPatterns`)
-   サポートされるケースでの暗黙的な返信先ボットの動作

`requireMention` はギルド/チャンネルごとに設定されます (`channels.discord.guilds...`)。`ignoreOtherMentions` は、ボット以外のユーザー/ロールにメンションしているがボットにはメンションしていないメッセージをオプションでドロップします (@everyone/@here を除く)。グループ DM:

-   デフォルト: 無視 (`dm.groupEnabled=false`)
-   `dm.groupChannels` 経由のオプションの許可リスト (チャンネル ID またはスラッグ)

### ロールベースのエージェントルーティング

`bindings[].match.roles` を使用して、Discord ギルドメンバーをロール ID ごとに異なるエージェントにルーティングします。ロールベースのバインディングはロール ID のみを受け付け、ピアまたは親ピアのバインディングの後、ギルドのみのバインディングの前に評価されます。バインディングが他のマッチフィールド (例: `peer` + `guildId` + `roles`) も設定する場合、設定されたすべてのフィールドが一致する必要があります。

```json
{
  bindings: [
    {
      agentId: "opus",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
        roles: ["111111111111111111"],
      },
    },
    {
      agentId: "sonnet",
      match: {
        channel: "discord",
        guildId: "123456789012345678",
      },
    },
  ],
}
```

## 開発者ポータル設定

1.  Discord Developer Portal -> **Applications** -> **New Application**
2.  **Bot** -> **Add Bot**
3.  ボットトークンをコピー

**Bot -> Privileged Gateway Intents** で以下を有効にします:

-   Message Content Intent
-   Server Members Intent (推奨)

プレゼンスインテントはオプションで、メンバーのプレゼンス更新を受け取りたい場合にのみ必要です。ボットのプレゼンス設定 (`setPresence`) は、メンバーのプレゼンス更新を有効にする必要はありません。

OAuth URL ジェネレーター:

-   スコープ: `bot`, `applications.commands`

典型的な基本権限:

-   View Channels
-   Send Messages
-   Read Message History
-   Embed Links
-   Attach Files
-   Add Reactions (オプション)

明示的に必要な場合を除き、`Administrator` は避けてください。

Discord 開発者モードを有効にし、以下をコピーします:

-   サーバー ID
-   チャンネル ID
-   ユーザー ID

信頼性のある監査とプローブのために、OpenClaw config では数値 ID を優先します。

## ネイティブコマンドとコマンド認証

-   `commands.native` はデフォルトで `"auto"` であり、Discord で有効になります。
-   チャンネルごとの上書き: `channels.discord.commands.native`。
-   `commands.native=false` は、以前に登録された Discord ネイティブコマンドを明示的にクリアします。
-   ネイティブコマンド認証は、通常のメッセージ処理と同じ Discord 許可リスト/ポリシーを使用します。
-   コマンドは、認可されていないユーザーに対しても Discord UI に表示される可能性がありますが、実行時には OpenClaw 認証が適用され、「認可されていません」が返されます。

コマンドカタログと動作については [スラッシュコマンド](../tools/slash-commands.md) を参照してください。デフォルトのスラッシュコマンド設定:

-   `ephemeral: true`

## 機能詳細

Discord はエージェント出力での返信タグをサポートしています:

-   `[[reply_to_current]]`
-   `[[reply_to:]]`

`channels.discord.replyToMode` によって制御されます:

-   `off` (デフォルト)
-   `first`
-   `all`

注: `off` は暗黙的な返信スレッド化を無効にします。明示的な `[[reply_to_*]]` タグは引き続き尊重されます。メッセージ ID はコンテキスト/履歴に表示されるため、エージェントは特定のメッセージをターゲットにできます。

OpenClaw は、一時メッセージを送信し、テキストが到着するにつれて編集することで、下書き返信をストリーミングできます。

-   `channels.discord.streaming` はプレビューストリーミングを制御します (`off` | `partial` | `block` | `progress`, デフォルト: `off`)。
-   `progress` はクロスチャンネル一貫性のために受け入れられ、Discord では `partial` にマッピングされます。
-   `channels.discord.streamMode` はレガシーエイリアスであり、自動移行されます。
-   `partial` はトークンが到着するにつれて単一のプレビューメッセージを編集します。
-   `block` は下書きサイズのチャンクを出力します (`draftChunk` を使用してサイズと改行ポイントを調整します)。

例:

```json
{
  channels: {
    discord: {
      streaming: "partial",
    },
  },
}
```

`block` モードのチャンキングデフォルト (`channels.discord.textChunkLimit` にクランプされます):

```json
{
  channels: {
    discord: {
      streaming: "block",
      draftChunk: {
        minChars: 200,
        maxChars: 800,
        breakPreference: "paragraph",
      },
    },
  },
}
```

プレビューストリーミングはテキストのみです。メディア返信は通常の配信にフォールバックします。注: プレビューストリーミングはブロックストリーミングとは別です。Discord に対してブロックストリーミングが明示的に有効になっている場合、OpenClaw は二重ストリーミングを避けるためにプレビューストリームをスキップします。

ギルド履歴コンテキスト:

-   `channels.discord.historyLimit` デフォルト `20`
-   フォールバック: `messages.groupChat.historyLimit`
-   `0` で無効

DM 履歴制御:

-   `channels.discord.dmHistoryLimit`
-   `channels.discord.dms["<user_id>"].historyLimit`

スレッド動作:

-   Discord スレッドはチャンネルセッションとしてルーティングされます
-   親スレッドメタデータは親セッションリンケージに使用できます
-   スレッド設定は、スレッド固有のエントリが存在しない限り、親チャンネル設定を継承します

チャンネルトピックは **信頼されていない** コンテキストとして注入されます (システムプロンプトとしては扱われません)。

Discord はスレッドをセッションターゲットにバインドできるため、そのスレッド内のフォローアップメッセージは同じセッション (サブエージェントセッションを含む) へのルーティングを維持します。コマンド:

-   `/focus ` 現在の/新しいスレッドをサブエージェント/セッションターゲットにバインド
-   `/unfocus` 現在のスレッドバインディングを削除
-   `/agents` アクティブな実行とバインディング状態を表示
-   `/session idle <duration|off>` フォーカスされたバインディングの非アクティブ自動アンフォーカスを検査/更新
-   `/session max-age <duration|off>` フォーカスされたバインディングのハード最大経過時間を検査/更新

設定:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        idleHours: 24,
        maxAgeHours: 0,
        spawnSubagentSessions: false, // オプトイン
      },
    },
  },
}
```

注記:

-   `session.threadBindings.*` はグローバルデフォルトを設定します。
-   `channels.discord.threadBindings.*` は Discord の動作を上書きします。
-   `spawnSubagentSessions` は `sessions_spawn({ thread: true })` に対してスレッドを自動作成/バインドするには true である必要があります。
-   `spawnAcpSessions` は ACP (`/acp spawn ... --thread ...` または `sessions_spawn({ runtime: "acp", thread: true })`) に対してスレッドを自動作成/バインドするには true である必要があります。
-   アカウントに対してスレッドバインディングが無効になっている場合、`/focus` および関連するスレッドバインディング操作は利用できません。

[サブエージェント](../tools/subagents.md)、[ACP エージェント](../tools/acp-agents.md)、および [設定リファレンス](../gateway/configuration-reference.md) を参照してください。

安定した「常時オン」の ACP ワークスペースの場合、Discord 会話をターゲットとするトップレベルの型付き ACP バインディングを設定します。設定パス:

-   `bindings[]` で `type: "acp"` および `match.channel: "discord"`

例:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": {
              requireMention: false,
            },
          },
        },
      },
    },
  },
}
```

注記:

-   スレッドメッセージは親チャンネルの ACP バインディングを継承できます。
-   バインドされたチャンネルまたはスレッド内では、`/new` および `/reset` は同じ ACP セッションをその場でリセットします。
-   一時的なスレッドバインディングも機能し、アクティブな間はターゲット解決を上書きできます。

バインディング動作の詳細については [ACP エージェント](../tools/acp-agents.md) を参照してください。

ギルドごとのリアクション通知モード:

-   `off`
-   `own` (デフォルト)
-   `all`
-   `allowlist` (`guilds..users` を使用)

リアクションイベントはシステムイベントに変換され、ルーティングされた Discord セッションに添付されます。

`ackReaction` は、OpenClaw が受信メッセージを処理している間に確認絵文字を送信します。解決順序:

-   `channels.discord.accounts..ackReaction`
-   `channels.discord.ackReaction`
-   `messages.ackReaction`
-   エージェントアイデンティティ絵文字フォールバック (`agents.list[].identity.emoji`, それ以外は ”👀”)

注記:

-   Discord は Unicode 絵文字またはカスタム絵文字名を受け付けます。
-   チャンネルまたはアカウントに対してリアクションを無効にするには `""` を使用します。

チャンネル開始の設定書き込みはデフォルトで有効です。これは `/config set|unset` フローに影響します (コマンド機能が有効な場合)。無効化:

```json
{
  channels: {
    discord: {
      configWrites: false,
    },
  },
}
```

Discord ゲートウェイ WebSocket トラフィックと起動時 REST ルックアップ (アプリケーション ID + 許可リスト解決) を `channels.discord.proxy` で HTTP(S) プロキシ経由でルーティングします。

```json
{
  channels: {
    discord: {
      proxy: "http://proxy.example:8080",
    },
  },
}
```

アカウントごとの上書き:

```json
{
  channels: {
    discord: {
      accounts: {
        primary: {
          proxy: "http://proxy.example:8080",
        },
      },
    },
  },
}
```

PluralKit 解決を有効にして、プロキシされたPluralKit 解決を有効にして、プロキシされたメッセージをシステムメンバーのIDにマップします:

```json
{
  channels: {
    discord: {
      pluralkit: {
        enabled: true,
        token: "pk_live_...", // オプション; プライベートシステムに必要
      },
    },
  },
}
```

注意:

-   許可リストは `pk:` を使用できます
-   メンバー表示名は `channels.discord.dangerouslyAllowNameMatching: true` の場合のみ名前/スラッグで一致します
-   検索は元のメッセージIDを使用し、時間枠で制限されています
-   検索に失敗した場合、プロキシされたメッセージはボットメッセージとして扱われ、`allowBots=true` 以外はドロップされます

ステータスまたはアクティビティフィールドを設定したとき、または自動プレゼンスを有効にしたときにプレゼンス更新が適用されます。ステータスのみの例:

```json
{
  channels: {
    discord: {
      status: "idle",
    },
  },
}
```

アクティビティ例（カスタムステータスはデフォルトのアクティビティタイプ）:

```json
{
  channels: {
    discord: {
      activity: "Focus time",
      activityType: 4,
    },
  },
}
```

ストリーミング例:

```json
{
  channels: {
    discord: {
      activity: "Live coding",
      activityType: 1,
      activityUrl: "https://twitch.tv/openclaw",
    },
  },
}
```

アクティビティタイプマップ:

-   0: プレイ中
-   1: ストリーミング中（`activityUrl` が必要）
-   2: リスニング中
-   3: ウォッチング中
-   4: カスタム（アクティビティテキストをステータス状態として使用; 絵文字はオプション）
-   5: 競争中

自動プレゼンス例（ランタイム健全性シグナル）:

```json
{
  channels: {
    discord: {
      autoPresence: {
        enabled: true,
        intervalMs: 30000,
        minUpdateIntervalMs: 15000,
        exhaustedText: "token exhausted",
      },
    },
  },
}
```

自動プレゼンスはランタイム可用性をDiscordステータスにマップします: 健全 => オンライン, 低下または不明 => アイドル, 不足または利用不可 => 取り込み中。オプションのテキストオーバーライド:

-   `autoPresence.healthyText`
-   `autoPresence.degradedText`
-   `autoPresence.exhaustedText` (`{reason}` プレースホルダーをサポート)

DiscordはDMでボタン ベースの実行承認をサポートし、オプションで元のチャンネルに承認プロンプトを投稿できます。構成パス:

-   `channels.discord.execApprovals.enabled`
-   `channels.discord.execApprovals.approvers`
-   `channels.discord.execApprovals.target` (`dm` | `channel` | `both`, デフォルト: `dm`)
-   `agentFilter`, `sessionFilter`, `cleanupAfterResolve`

`target` が `channel` または `both` の場合、承認プロンプトはチャンネルで表示されます。構成された承認者のみがボタンを使用でき、他のユーザーは一時的な拒否を受け取ります。承認プロンプトにはコマンドテキストが含まれているため、信頼できるチャンネルでのみチャンネル配信を有効にしてください。セッションキーからチャンネルIDを導出できない場合、OpenClawはDM配信にフォールバックします。

不明な承認IDで承認が失敗する場合は、承認者リストと機能有効化を確認してください。

関連ドキュメント: [実行承認](../tools/exec-approvals.md)

## ツールとアクションゲート

## 安全性と運用

-   ボットトークンをシークレットとして扱います（監視環境では `DISCORD_BOT_TOKEN` が推奨）。
-   最小権限のDiscord権限を付与します。
-   コマンドのデプロイ/ステートが古い場合は、ゲートウェイを再起動して `openclaw channels status --probe` で再確認してください。

## 関連

-   [ペアリング](./pairing.md)
-   [チャンネルルーティング](./channel-routing.md)
-   [マルチエージェントルーティング](../concepts/multi-agent.md)
-   [トラブルシューティング](./troubleshooting.md)
-   [スラッシュコマンド](../tools/slash-commands.md)
