

  メッセージングプラットフォーム

  
# Feishu

Feishu（Lark）は、企業がメッセージングとコラボレーションに使用するチームチャットプラットフォームです。このプラグインは、Feishu/LarkのWebSocketイベントサブスクリプションを使用してOpenClawをFeishu/Larkボットに接続し、公開Webhook URLを公開せずにメッセージを受信できるようにします。

* * *

## 必要なプラグイン

Feishuプラグインをインストールします：

```bash
openclaw plugins install @openclaw/feishu
```

ローカルチェックアウト（gitリポジトリから実行する場合）：

```bash
openclaw plugins install ./extensions/feishu
```

* * *

## クイックスタート

Feishuチャネルを追加する方法は2つあります：

### 方法1: オンボーディングウィザード（推奨）

OpenClawをインストールしたばかりの場合は、ウィザードを実行します：

```bash
openclaw onboard
```

ウィザードは以下の手順を案内します：

1.  Feishuアプリの作成と認証情報の収集
2.  OpenClawでのアプリ認証情報の設定
3.  ゲートウェイの起動

✅ **設定後**、ゲートウェイの状態を確認します：

-   `openclaw gateway status`
-   `openclaw logs --follow`

### 方法2: CLIセットアップ

初期インストールが完了している場合は、CLI経由でチャネルを追加します：

```bash
openclaw channels add
```

**Feishu**を選択し、App IDとApp Secretを入力します。✅ **設定後**、ゲートウェイを管理します：

-   `openclaw gateway status`
-   `openclaw gateway restart`
-   `openclaw logs --follow`

* * *

## ステップ1: Feishuアプリを作成する

### 1\. Feishuオープンプラットフォームを開く

[Feishuオープンプラットフォーム](https://open.feishu.cn/app)にアクセスしてサインインします。Lark（グローバル）テナントは[https://open.larksuite.com/app](https://open.larksuite.com/app)を使用し、Feishu設定で`domain: "lark"`を設定する必要があります。

### 2\. アプリを作成する

1.  **企業アプリを作成**をクリック
2.  アプリ名と説明を入力
3.  アプリアイコンを選択

![企業アプリを作成](../images/channels-feishu-step2-create-app.png.md)

### 3\. 認証情報をコピーする

**認証情報と基本情報**から以下をコピーします：

-   **App ID**（形式：`cli_xxx`）
-   **App Secret**

❗ **重要：** App Secretは秘密にしてください。 ![認証情報を取得](../images/channels-feishu-step3-credentials.png.md)

### 4\. 権限を設定する

**権限**で、**一括インポート**をクリックし、以下を貼り付けます：

```json
{
  "scopes": {
    "tenant": [
      "aily:file:read",
      "aily:file:write",
      "application:application.app_message_stats.overview:readonly",
      "application:application:self_manage",
      "application:bot.menu:write",
      "cardkit:card:read",
      "cardkit:card:write",
      "contact:user.employee_id:readonly",
      "corehr:file:download",
      "event:ip_list",
      "im:chat.access_event.bot_p2p_chat:read",
      "im:chat.members:bot_access",
      "im:message",
      "im:message.group_at_msg:readonly",
      "im:message.p2p_msg:readonly",
      "im:message:readonly",
      "im:message:send_as_bot",
      "im:resource"
    ],
    "user": ["aily:file:read", "aily:file:write", "im:chat.access_event.bot_p2p_chat:read"]
  }
}
```

![権限を設定](../images/channels-feishu-step4-permissions.png.md)

### 5\. ボット機能を有効化する

**アプリ機能** > **ボット**で：

1.  ボット機能を有効化
2.  ボット名を設定

![ボット機能を有効化](../images/channels-feishu-step5-bot-capability.png.md)

### 6\. イベントサブスクリプションを設定する

⚠️ **重要：** イベントサブスクリプションを設定する前に、以下を確認してください：

1.  Feishuに対して`openclaw channels add`を実行済みであること
2.  ゲートウェイが実行中であること（`openclaw gateway status`）

**イベントサブスクリプション**で：

1.  **長い接続を使用してイベントを受信する**（WebSocket）を選択
2.  イベント`im.message.receive_v1`を追加

⚠️ ゲートウェイが実行されていない場合、長い接続の設定は保存に失敗する可能性があります。 ![イベントサブスクリプションを設定](../images/channels-feishu-step6-event-subscription.png.md)

### 7\. アプリを公開する

1.  **バージョン管理とリリース**でバージョンを作成
2.  レビューに提出して公開
3.  管理者の承認を待つ（企業アプリは通常自動承認されます）

* * *

## ステップ2: OpenClawを設定する

### ウィザードで設定する（推奨）

```bash
openclaw channels add
```

**Feishu**を選択し、App IDとApp Secretを貼り付けます。

### 設定ファイルで設定する

`~/.openclaw/openclaw.json`を編集します：

```json
{
  channels: {
    feishu: {
      enabled: true,
      dmPolicy: "pairing",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "My AI assistant",
        },
      },
    },
  },
}
```

`connectionMode: "webhook"`を使用する場合は、`verificationToken`を設定します。Feishu Webhookサーバーはデフォルトで`127.0.0.1`にバインドされます。異なるバインドアドレスを意図的に必要とする場合のみ`webhookHost`を設定してください。

#### 検証トークン（Webhookモード）

Webhookモードを使用する場合は、設定で`channels.feishu.verificationToken`を設定します。値を取得するには：

1.  Feishuオープンプラットフォームでアプリを開く
2.  **開発** → **イベントとコールバック**に移動
3.  **暗号化**タブを開く
4.  **検証トークン**をコピー

![検証トークンの場所](../images/channels-feishu-verification-token.png.md)

### 環境変数で設定する

```bash
export FEISHU_APP_ID="cli_xxx"
export FEISHU_APP_SECRET="xxx"
```

### Lark（グローバル）ドメイン

テナントがLark（国際版）にある場合は、ドメインを`lark`（または完全なドメイン文字列）に設定します。`channels.feishu.domain`またはアカウントごとに（`channels.feishu.accounts..domain`）設定できます。

```json
{
  channels: {
    feishu: {
      domain: "lark",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
        },
      },
    },
  },
}
```

### クォータ最適化フラグ

2つのオプションフラグでFeishu APIの使用量を削減できます：

-   `typingIndicator`（デフォルト`true`）：`false`の場合、タイピングリアクション呼び出しをスキップします。
-   `resolveSenderNames`（デフォルト`true`）：`false`の場合、送信者プロファイル検索呼び出しをスキップします。

トップレベルまたはアカウントごとに設定します：

```json
{
  channels: {
    feishu: {
      typingIndicator: false,
      resolveSenderNames: false,
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          typingIndicator: true,
          resolveSenderNames: false,
        },
      },
    },
  },
}
```

* * *

## ステップ3: 起動 + テスト

### 1\. ゲートウェイを起動する

```bash
openclaw gateway
```

### 2\. テストメッセージを送信する

Feishuでボットを見つけ、メッセージを送信します。

### 3\. ペアリングを承認する

デフォルトでは、ボットはペアリングコードで応答します。承認します：

```bash
openclaw pairing approve feishu <CODE>
```

承認後、通常通りチャットできます。

* * *

## 概要

-   **Feishuボットチャネル**：ゲートウェイによって管理されるFeishuボット
-   **決定論的ルーティング**：返信は常にFeishuに戻ります
-   **セッション分離**：DMはメインセッションを共有し、グループは分離されます
-   **WebSocket接続**：Feishu SDK経由の長い接続、公開URLは不要

* * *

## アクセス制御

### ダイレクトメッセージ

-   **デフォルト**：`dmPolicy: "pairing"`（不明なユーザーはペアリングコードを受け取ります）
-   **ペアリングを承認**：
    
    コピー
    
    ```bash
    openclaw pairing list feishu
    openclaw pairing approve feishu <CODE>
    ```
    
-   **許可リストモード**：`channels.feishu.allowFrom`に許可されたOpen IDを設定

### グループチャット

**1\. グループポリシー**（`channels.feishu.groupPolicy`）：

-   `"open"` = グループ内の全員を許可（デフォルト）
-   `"allowlist"` = `groupAllowFrom`のみ許可
-   `"disabled"` = グループメッセージを無効化

**2\. メンション要件**（`channels.feishu.groups.<chat_id>.requireMention`）：

-   `true` = @メンションを必要とする（デフォルト）
-   `false` = メンションなしで応答

* * *

## グループ設定例

### すべてのグループを許可、@メンションが必要（デフォルト）

```json
{
  channels: {
    feishu: {
      groupPolicy: "open",
      // デフォルト requireMention: true
    },
  },
}
```

### すべてのグループを許可、@メンションは不要

```json
{
  channels: {
    feishu: {
      groups: {
        oc_xxx: { requireMention: false },
      },
    },
  },
}
```

### 特定のグループのみ許可

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      // FeishuグループID（chat_id）は oc_xxx のようになります
      groupAllowFrom: ["oc_xxx", "oc_yyy"],
    },
  },
}
```

### グループ内でメッセージを送信できる送信者を制限する（送信者許可リスト）

グループ自体を許可することに加えて、**そのグループ内のすべてのメッセージ**は送信者のopen_idによってゲートされます：`groups.<chat_id>.allowFrom`にリストされたユーザーのみがメッセージを処理され、他のメンバーからのメッセージは無視されます（これは、/resetや/newなどの制御コマンドのみではなく、送信者レベルの完全なゲーティングです）。

```json
{
  channels: {
    feishu: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["oc_xxx"],
      groups: {
        oc_xxx: {
          // FeishuユーザーID（open_id）は ou_xxx のようになります
          allowFrom: ["ou_user1", "ou_user2"],
        },
      },
    },
  },
}
```

* * *

## グループ/ユーザーIDを取得する

### グループID（chat_id）

グループIDは`oc_xxx`のようになります。**方法1（推奨）**

1.  ゲートウェイを起動し、グループ内でボットを@メンションします
2.  `openclaw logs --follow`を実行し、`chat_id`を探します

**方法2** Feishu APIデバッガーを使用してグループチャットをリストします。

### ユーザーID（open_id）

ユーザーIDは`ou_xxx`のようになります。**方法1（推奨）**

1.  ゲートウェイを起動し、ボットにDMを送信します
2.  `openclaw logs --follow`を実行し、`open_id`を探します

**方法2** ペアリングリクエストでユーザーOpen IDを確認します：

```bash
openclaw pairing list feishu
```

* * *

## 一般的なコマンド

| コマンド | 説明 |
| --- | --- |
| `/status` | ボットの状態を表示 |
| `/reset` | セッションをリセット |
| `/model` | モデルを表示/切り替え |

> 注：Feishuはネイティブのコマンドメニューをまだサポートしていないため、コマンドはテキストとして送信する必要があります。

## ゲートウェイ管理コマンド

| コマンド | 説明 |
| --- | --- |
| `openclaw gateway status` | ゲートウェイの状態を表示 |
| `openclaw gateway install` | ゲートウェイサービスをインストール/起動 |
| `openclaw gateway stop` | ゲートウェイサービスを停止 |
| `openclaw gateway restart` | ゲートウェイサービスを再起動 |
| `openclaw logs --follow` | ゲートウェイログを追跡 |

* * *

## トラブルシューティング

### ボットがグループチャットで応答しない

1.  ボットがグループに追加されていることを確認
2.  ボットを@メンションしていることを確認（デフォルト動作）
3.  `groupPolicy`が`"disabled"`に設定されていないことを確認
4.  ログを確認：`openclaw logs --follow`

### ボットがメッセージを受信しない

1.  アプリが公開され承認されていることを確認
2.  イベントサブスクリプションに`im.message.receive_v1`が含まれていることを確認
3.  **長い接続**が有効になっていることを確認
4.  アプリの権限が完全であることを確認
5.  ゲートウェイが実行中であることを確認：`openclaw gateway status`
6.  ログを確認：`openclaw logs --follow`

### App Secretの漏洩

1.  FeishuオープンプラットフォームでApp Secretをリセット
2.  設定内のApp Secretを更新
3.  ゲートウェイを再起動

### メッセージ送信失敗

1.  アプリに`im:message:send_as_bot`権限があることを確認
2.  アプリが公開されていることを確認
3.  詳細なエラーについてログを確認

* * *

## 高度な設定

### 複数アカウント

```json
{
  channels: {
    feishu: {
      defaultAccount: "main",
      accounts: {
        main: {
          appId: "cli_xxx",
          appSecret: "xxx",
          botName: "Primary bot",
        },
        backup: {
          appId: "cli_yyy",
          appSecret: "yyy",
          botName: "Backup bot",
          enabled: false,
        },
      },
    },
  },
}
```

`defaultAccount`は、アウトバウンドAPIが明示的に`accountId`を指定しない場合に使用されるFeishuアカウントを制御します。

### メッセージ制限

-   `textChunkLimit`：アウトバウンドテキストのチャンクサイズ（デフォルト：2000文字）
-   `mediaMaxMb`：メディアアップロード/ダウンロード制限（デフォルト：30MB）

### ストリーミング

Feishuはインタラクティブカードを介したストリーミング返信をサポートしています。有効にすると、ボットはテキストを生成しながらカードを更新します。

```json
{
  channels: {
    feishu: {
      streaming: true, // ストリーミングカード出力を有効化（デフォルト true）
      blockStreaming: true, // ブロックレベルのストリーミングを有効化（デフォルト true）
    },
  },
}
```

`streaming: false`を設定すると、完全な返信を待ってから送信します。

### マルチエージェントルーティング

`bindings`を使用して、FeishuのDMまたはグループを異なるエージェントにルーティングします。

```json
{
  agents: {
    list: [
      { id: "main" },
      {
        id: "clawd-fan",
        workspace: "/home/user/clawd-fan",
        agentDir: "/home/user/.openclaw/agents/clawd-fan/agent",
      },
      {
        id: "clawd-xi",
        workspace: "/home/user/clawd-xi",
        agentDir: "/home/user/.openclaw/agents/clawd-xi/agent",
      },
    ],
  },
  bindings: [
    {
      agentId: "main",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_xxx" },
      },
    },
    {
      agentId: "clawd-fan",
      match: {
        channel: "feishu",
        peer: { kind: "direct", id: "ou_yyy" },
      },
    },
    {
      agentId: "clawd-xi",
      match: {
        channel: "feishu",
        peer: { kind: "group", id: "oc_zzz" },
      },
    },
  ],
}
```

ルーティングフィールド：

-   `match.channel`：`"feishu"`
-   `match.peer.kind`：`"direct"`または`"group"`
-   `match.peer.id`：ユーザーOpen ID（`ou_xxx`）またはグループID（`oc_xxx`）

検索のヒントについては[グループ/ユーザーIDを取得する](#get-groupuser-ids)を参照してください。

* * *

## 設定リファレンス

完全な設定：[ゲートウェイ設定](../gateway/configuration.md) 主なオプション：

| 設定 | 説明 | デフォルト |
| --- | --- | --- |
| `channels.feishu.enabled` | チャネルの有効/無効 | `true` |
| `channels.feishu.domain` | APIドメイン（`feishu`または`lark`） | `feishu` |
| `channels.feishu.connectionMode` | イベント転送モード | `websocket` |
| `channels.feishu.defaultAccount` | アウトバウンドルーティングのデフォルトアカウントID | `default` |
| `channels.feishu.verificationToken` | Webhookモードで必須 | \- |
| `channels.feishu.webhookPath` | Webhookルートパス | `/feishu/events` |
| `channels.feishu.webhookHost` | Webhookバインドホスト | `127.0.0.1` |
| `channels.feishu.webhookPort` | Webhookバインドポート | `3000` |
| `channels.feishu.accounts..appId` | App ID | \- |
| `channels.feishu.accounts..appSecret` | App Secret | \- |
| `channels.feishu.accounts..domain` | アカウントごとのAPIドメイン上書き | `feishu` |
| `channels.feishu.dmPolicy` | DMポリシー | `pairing` |
| `channels.feishu.allowFrom` | DM許可リスト（open_idリスト） | \- |
| `channels.feishu.groupPolicy` | グループポリシー | `open` |
| `channels.feishu.groupAllowFrom` | グループ許可リスト | \- |
| `channels.feishu.groups.<chat_id>.requireMention` | @メンションを必要とする | `true` |
| `channels.feishu.groups.<chat_id>.enabled` | グループを有効化 | `true` |
| `channels.feishu.textChunkLimit` | メッセージチャンクサイズ | `2000` |
| `channels.feishu.mediaMaxMb` | メディアサイズ制限 | `30` |
| `channels.feishu.streaming` | ストリーミングカード出力を有効化 | `true` |
| `channels.feishu.blockStreaming` | ブロックストリーミングを有効化 | `true` |

* * *

## dmPolicyリファレンス

| 値 | 動作 |
| --- | --- |
| `"pairing"` | **デフォルト。** 不明なユーザーはペアリングコードを受け取り、承認が必要 |
| `"allowlist"` | `allowFrom`内のユーザーのみチャット可能 |
| `"open"` | すべてのユーザーを許可（`allowFrom`に`"*"`が必要） |
| `"disabled"` | DMを無効化 |

* * *

## サポートされているメッセージタイプ

### 受信

-   ✅ テキスト
-   ✅ リッチテキスト（投稿）
-   ✅ 画像
-   ✅ ファイル
-   ✅ 音声
-   ✅ 動画
-   ✅ ステッカー

### 送信

-   ✅ テキスト
-   ✅ 画像
-   ✅ ファイル
-   ✅ 音声
-   ⚠️ リッチテキスト（部分的なサポート）

[Discord](./discord.md)[Google Chat](./googlechat.md)