

  マルチエージェント

  
# マルチエージェントルーティング

目標: 1つの稼働中のGatewayで、複数の*分離された*エージェント（別々のワークスペース + `agentDir` + セッション）と、複数のチャネルアカウント（例: 2つのWhatsApp）を実現すること。インバウンドメッセージはバインディングを介してエージェントにルーティングされます。

## 「1つのエージェント」とは？

**エージェント**は、以下を独自に持つ完全にスコープされた「脳」です:

-   **ワークスペース** (ファイル、AGENTS.md/SOUL.md/USER.md、ローカルノート、ペルソナルール)。
-   **状態ディレクトリ** (`agentDir`) - 認証プロファイル、モデルレジストリ、エージェントごとの設定用。
-   **セッションストア** (チャット履歴 + ルーティング状態) - `~/.openclaw/agents//sessions` 以下。

認証プロファイルは**エージェントごと**です。各エージェントは自身の以下から読み込みます:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

メインエージェントの認証情報は**自動的には共有されません**。エージェント間で `agentDir` を再利用しないでください（認証/セッションの衝突を引き起こします）。認証情報を共有したい場合は、`auth-profiles.json` を他のエージェントの `agentDir` にコピーしてください。スキルは各ワークスペースの `skills/` フォルダを介してエージェントごとに設定され、共有スキルは `~/.openclaw/skills` から利用可能です。詳細は [スキル: エージェントごと vs 共有](../tools/skills.md#per-agent-vs-shared-skills) を参照してください。Gatewayは**1つのエージェント**（デフォルト）または**複数のエージェント**を並行してホストできます。**ワークスペースに関する注意:** 各エージェントのワークスペースは**デフォルトのカレントワーキングディレクトリ**であり、厳密なサンドボックスではありません。相対パスはワークスペース内で解決されますが、絶対パスはサンドボックスが有効でない限り、ホスト上の他の場所にアクセスする可能性があります。詳細は [サンドボックス化](../gateway/sandboxing.md) を参照してください。

## パス（クイックマップ）

-   設定: `~/.openclaw/openclaw.json` (または `OPENCLAW_CONFIG_PATH`)
-   状態ディレクトリ: `~/.openclaw` (または `OPENCLAW_STATE_DIR`)
-   ワークスペース: `~/.openclaw/workspace` (または `~/.openclaw/workspace-`)
-   エージェントディレクトリ: `~/.openclaw/agents//agent` (または `agents.list[].agentDir`)
-   セッション: `~/.openclaw/agents//sessions`

### シングルエージェントモード（デフォルト）

何もしない場合、OpenClawは単一のエージェントを実行します:

-   `agentId` はデフォルトで **`main`** になります。
-   セッションは `agent:main:` としてキー付けされます。
-   ワークスペースはデフォルトで `~/.openclaw/workspace` になります (`OPENCLAW_PROFILE` が設定されている場合は `~/.openclaw/workspace-`)。
-   状態はデフォルトで `~/.openclaw/agents/main/agent` になります。

## エージェントヘルパー

エージェントウィザードを使用して、新しい分離エージェントを追加します:

```bash
openclaw agents add work
```

次に、インバウンドメッセージをルーティングするために `bindings` を追加します（またはウィザードに任せます）。以下で確認します:

```bash
openclaw agents list --bindings
```

## クイックスタート

### ステップ 1: 各エージェントのワークスペースを作成

ウィザードを使用するか、手動でワークスペースを作成します:

```bash
openclaw agents add coding
openclaw agents add social
```

各エージェントは、`SOUL.md`、`AGENTS.md`、オプションで `USER.md` を含む独自のワークスペースと、専用の `agentDir`、および `~/.openclaw/agents/` 以下のセッションストアを取得します。

### ステップ 2: チャネルアカウントを作成

希望のチャネルでエージェントごとに1つのアカウントを作成します:

-   Discord: エージェントごとに1つのボット、Message Content Intentを有効化、各トークンをコピー。
-   Telegram: BotFather経由でエージェントごとに1つのボット、各トークンをコピー。
-   WhatsApp: アカウントごとに各電話番号をリンク。

```bash
openclaw channels login --channel whatsapp --account work
```

チャネルガイドを参照: [Discord](../channels/discord.md), [Telegram](../channels/telegram.md), [WhatsApp](../channels/whatsapp.md)。

### ステップ 3: エージェント、アカウント、バインディングを追加

`agents.list` の下にエージェントを、`channels..accounts` の下にチャネルアカウントを追加し、`bindings` でそれらを接続します（以下の例を参照）。

### ステップ 4: 再起動して確認

```bash
openclaw gateway restart
openclaw agents list --bindings
openclaw channels status --probe
```

## 複数エージェント = 複数の人物、複数の人格

**複数のエージェント**を使用すると、各 `agentId` は**完全に分離されたペルソナ**になります:

-   **異なる電話番号/アカウント** (チャネルごとの `accountId`)。
-   **異なる人格** (エージェントごとのワークスペースファイル、例: `AGENTS.md` と `SOUL.md`)。
-   **分離された認証 + セッション** (明示的に有効にしない限り、クロストークなし)。

これにより、**複数の人物**が1つのGatewayサーバーを共有しながら、各自のAI「脳」とデータを分離して保持できます。

## 1つのWhatsApp番号、複数の人物（DM分割）

**1つのWhatsAppアカウント**に留まりながら、**異なるWhatsApp DM**を異なるエージェントにルーティングできます。送信者E.164（例: `+15551234567`）と `peer.kind: "direct"` でマッチングします。返信は依然として同じWhatsApp番号から送信されます（エージェントごとの送信者識別情報はありません）。重要な詳細: ダイレクトチャットはエージェントの**メインセッションキー**に統合されるため、真の分離には**人物ごとに1つのエージェント**が必要です。例:

```json
{
  agents: {
    list: [
      { id: "alex", workspace: "~/.openclaw/workspace-alex" },
      { id: "mia", workspace: "~/.openclaw/workspace-mia" },
    ],
  },
  bindings: [
    {
      agentId: "alex",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230001" } },
    },
    {
      agentId: "mia",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551230002" } },
    },
  ],
  channels: {
    whatsapp: {
      dmPolicy: "allowlist",
      allowFrom: ["+15551230001", "+15551230002"],
    },
  },
}
```

注意点:

-   DMアクセス制御は**WhatsAppアカウントごとにグローバル**です（ペアリング/許可リスト）。エージェントごとではありません。
-   共有グループの場合は、グループを1つのエージェントにバインドするか、[ブロードキャストグループ](../channels/broadcast-groups.md)を使用してください。

## ルーティングルール（メッセージがエージェントを選択する方法）

バインディングは**決定的**で、**最も具体的なものが優先**されます:

1.  `peer` マッチ (正確なDM/グループ/チャネルID)
2.  `parentPeer` マッチ (スレッド継承)
3.  `guildId + roles` (Discordロールルーティング)
4.  `guildId` (Discord)
5.  `teamId` (Slack)
6.  チャネルに対する `accountId` マッチ
7.  チャネルレベルのマッチ (`accountId: "*"`)
8.  デフォルトエージェントへのフォールバック (`agents.list[].default`、なければリストの最初のエントリ、デフォルト: `main`)

同じ階層で複数のバインディングがマッチした場合、設定順序で最初のものが優先されます。バインディングが複数のマッチフィールドを設定する場合（例: `peer` + `guildId`）、指定されたすべてのフィールドが必要です（`AND` セマンティクス）。アカウントスコープに関する重要な詳細:

-   `accountId` を省略したバインディングは、デフォルトアカウントのみにマッチします。
-   すべてのアカウントにわたるチャネル全体のフォールバックには `accountId: "*"` を使用します。
-   後で同じエージェントに対して明示的なアカウントIDで同じバインディングを追加すると、OpenClawは既存のチャネル専用バインディングを複製するのではなく、アカウントスコープにアップグレードします。

## 複数アカウント / 電話番号

**複数アカウント**をサポートするチャネル（例: WhatsApp）は、各ログインを識別するために `accountId` を使用します。各 `accountId` は異なるエージェントにルーティングできるため、1つのサーバーでセッションを混在させることなく複数の電話番号をホストできます。`accountId` が省略された場合のチャネル全体のデフォルトアカウントを設定したい場合は、`channels..defaultAccount` を設定します（オプション）。設定されていない場合、OpenClawは `default` が存在すればそれに、なければ最初に設定されたアカウントID（ソート順）にフォールバックします。このパターンをサポートする一般的なチャネルには以下が含まれます:

-   `whatsapp`, `telegram`, `discord`, `slack`, `signal`, `imessage`
-   `irc`, `line`, `googlechat`, `mattermost`, `matrix`, `nextcloud-talk`
-   `bluebubbles`, `zalo`, `zalouser`, `nostr`, `feishu`

## 概念

-   `agentId`: 1つの「脳」（ワークスペース、エージェントごとの認証、エージェントごとのセッションストア）。
-   `accountId`: 1つのチャネルアカウントインスタンス（例: WhatsAppアカウント `"personal"` と `"biz"`）。
-   `binding`: `(channel, accountId, peer)` およびオプションでギルド/チームIDによって、インバウンドメッセージを `agentId` にルーティングします。
-   ダイレクトチャットは `agent::` に統合されます（エージェントごとの「メイン」; `session.mainKey`）。

## プラットフォーム例

### エージェントごとのDiscordボット

各Discordボットアカウントは一意の `accountId` にマッピングされます。各アカウントをエージェントにバインドし、ボットごとに許可リストを保持します。

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "coding", workspace: "~/.openclaw/workspace-coding" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "discord", accountId: "default" } },
    { agentId: "coding", match: { channel: "discord", accountId: "coding" } },
  ],
  channels: {
    discord: {
      groupPolicy: "allowlist",
      accounts: {
        default: {
          token: "DISCORD_BOT_TOKEN_MAIN",
          guilds: {
            "123456789012345678": {
              channels: {
                "222222222222222222": { allow: true, requireMention: false },
              },
            },
          },
        },
        coding: {
          token: "DISCORD_BOT_TOKEN_CODING",
          guilds: {
            "123456789012345678": {
              channels: {
                "333333333333333333": { allow: true, requireMention: false },
              },
            },
          },
        },
      },
    },
  },
}
```

注意点:

-   各ボットをギルドに招待し、Message Content Intentを有効にします。
-   トークンは `channels.discord.accounts..token` に保存されます（デフォルトアカウントは `DISCORD_BOT_TOKEN` を使用できます）。

### エージェントごとのTelegramボット

```json
{
  agents: {
    list: [
      { id: "main", workspace: "~/.openclaw/workspace-main" },
      { id: "alerts", workspace: "~/.openclaw/workspace-alerts" },
    ],
  },
  bindings: [
    { agentId: "main", match: { channel: "telegram", accountId: "default" } },
    { agentId: "alerts", match: { channel: "telegram", accountId: "alerts" } },
  ],
  channels: {
    telegram: {
      accounts: {
        default: {
          botToken: "123456:ABC...",
          dmPolicy: "pairing",
        },
        alerts: {
          botToken: "987654:XYZ...",
          dmPolicy: "allowlist",
          allowFrom: ["tg:123456789"],
        },
      },
    },
  },
}
```

注意点:

-   BotFatherでエージェントごとに1つのボットを作成し、各トークンをコピーします。
-   トークンは `channels.telegram.accounts..botToken` に保存されます（デフォルトアカウントは `TELEGRAM_BOT_TOKEN` を使用できます）。

### エージェントごとのWhatsApp番号

Gatewayを起動する前に各アカウントをリンクします:

```bash
openclaw channels login --channel whatsapp --account personal
openclaw channels login --channel whatsapp --account biz
```

`~/.openclaw/openclaw.json` (JSON5):

```json
{
  agents: {
    list: [
      {
        id: "home",
        default: true,
        name: "Home",
        workspace: "~/.openclaw/workspace-home",
        agentDir: "~/.openclaw/agents/home/agent",
      },
      {
        id: "work",
        name: "Work",
        workspace: "~/.openclaw/workspace-work",
        agentDir: "~/.openclaw/agents/work/agent",
      },
    ],
  },

  // 決定的ルーティング: 最初のマッチが優先（最も具体的なものから順に）。
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },

    // オプションのピアごとのオーバーライド（例: 特定のグループをworkエージェントに送信）。
    {
      agentId: "work",
      match: {
        channel: "whatsapp",
        accountId: "personal",
        peer: { kind: "group", id: "1203630...@g.us" },
      },
    },
  ],

  // デフォルトではオフ: エージェント間メッセージングは明示的に有効化 + 許可リスト登録が必要。
  tools: {
    agentToAgent: {
      enabled: false,
      allow: ["home", "work"],
    },
  },

  channels: {
    whatsapp: {
      accounts: {
        personal: {
          // オプションのオーバーライド。デフォルト: ~/.openclaw/credentials/whatsapp/personal
          // authDir: "~/.openclaw/credentials/whatsapp/personal",
        },
        biz: {
          // オプションのオーバーライド。デフォルト: ~/.openclaw/credentials/whatsapp/biz
          // authDir: "~/.openclaw/credentials/whatsapp/biz",
        },
      },
    },
  },
}
```

## 例: WhatsApp日常チャット + Telegram深い作業

チャネルで分割: WhatsAppを高速な日常エージェントに、TelegramをOpusエージェントにルーティングします。

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    { agentId: "chat", match: { channel: "whatsapp" } },
    { agentId: "opus", match: { channel: "telegram" } },
  ],
}
```

注意点:

-   チャネルに複数のアカウントがある場合は、バインディングに `accountId` を追加します（例: `{ channel: "whatsapp", accountId: "personal" }`）。
-   残りをchatエージェントに保ちながら、単一のDM/グループをOpusにルーティングするには、そのピアに対して `match.peer` バインディングを追加します。ピアマッチは常にチャネル全体のルールよりも優先されます。

## 例: 同じチャネル、1つのピアをOpusに

WhatsAppを高速エージェントに保ちつつ、1つのDMをOpusにルーティング:

```json
{
  agents: {
    list: [
      {
        id: "chat",
        name: "Everyday",
        workspace: "~/.openclaw/workspace-chat",
        model: "anthropic/claude-sonnet-4-5",
      },
      {
        id: "opus",
        name: "Deep Work",
        workspace: "~/.openclaw/workspace-opus",
        model: "anthropic/claude-opus-4-6",
      },
    ],
  },
  bindings: [
    {
      agentId: "opus",
      match: { channel: "whatsapp", peer: { kind: "direct", id: "+15551234567" } },
    },
    { agentId: "chat", match: { channel: "whatsapp" } },
  ],
}
```

ピアバインディングは常に優先されるため、チャネル全体のルールよりも上に配置します。

## WhatsAppグループにバインドされた家族エージェント

専用の家族エージェントを単一のWhatsAppグループにバインドし、メンションゲーティングと厳格なツールポリシーを設定:

```json
{
  agents: {
    list: [
      {
        id: "family",
        name: "Family",
        workspace: "~/.openclaw/workspace-family",
        identity: { name: "Family Bot" },
        groupChat: {
          mentionPatterns: ["@family", "@familybot", "@Family Bot"],
        },
        sandbox: {
          mode: "all",
          scope: "agent",
        },
        tools: {
          allow: [
            "exec",
            "read",
            "sessions_list",
            "sessions_history",
            "sessions_send",
            "sessions_spawn",
            "session_status",
          ],
          deny: ["write", "edit", "apply_patch", "browser", "canvas", "nodes", "cron"],
        },
      },
    ],
  },
  bindings: [
    {
      agentId: "family",
      match: {
        channel: "whatsapp",
        peer: { kind: "group", id: "120363999999999999@g.us" },
      },
    },
  ],
}
```

注意点:

-   ツールの許可/拒否リストは**ツール**であり、スキルではありません。スキルがバイナリを実行する必要がある場合は、`exec` が許可されていることと、サンドボックス内にバイナリが存在することを確認してください。
-   より厳格なゲーティングのためには、`agents.list[].groupChat.mentionPatterns` を設定し、チャネルのグループ許可リストを有効に保ちます。

## エージェントごとのサンドボックスとツール設定

v2026.1.6以降、各エージェントは独自のサンドボックスとツール制限を持つことができます:

```json
{
  agents: {
    list: [
      {
        id: "personal",
        workspace: "~/.openclaw/workspace-personal",
        sandbox: {
          mode: "off",  // 個人エージェントにはサンドボックスなし
        },
        // ツール制限なし - すべてのツールが利用可能
      },
      {
        id: "family",
        workspace: "~/.openclaw/workspace-family",
        sandbox: {
          mode: "all",     // 常にサンドボックス化
          scope: "agent",  // エージェントごとに1つのコンテナ
          docker: {
            // コンテナ作成後のオプションのワンタイムセットアップ
            setupCommand: "apt-get update && apt-get install -y git curl",
          },
        },
        tools: {
          allow: ["read"],                    // readツールのみ許可
          deny: ["exec", "write", "edit", "apply_patch"],    // その他は拒否
        },
      },
    ],
  },
}
```

注意: `setupCommand` は `sandbox.docker` の下にあり、コンテナ作成時に1回実行されます。解決されたスコープが `"shared"` の場合、エージェントごとの `sandbox.docker.*` オーバーライドは無視されます。**利点:**

-   **セキュリティ分離**: 信頼されていないエージェントのツールを制限
-   **リソース制御**: 特定のエージェントをサンドボックス化しつつ、他をホスト上に維持
-   **柔軟なポリシー**: エージェントごとに異なる権限

注意: `tools.elevated` は**グローバル**で送信者ベースです。エージェントごとに設定することはできません。エージェントごとの境界が必要な場合は、`exec` を拒否するために `agents.list[].tools` を使用してください。グループターゲティングの場合は、`agents.list[].groupChat.mentionPatterns` を使用して、@メンションが意図したエージェントにクリーンにマッピングされるようにします。詳細な例については [マルチエージェントサンドボックス & ツール](../tools/multi-agent-sandbox-tools.md) を参照してください。

[圧縮](./compaction.md)[プレゼンス](./presence.md)