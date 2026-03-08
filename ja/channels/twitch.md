

  メッセージングプラットフォーム

  
# Twitch

IRC接続によるTwitchチャットサポート。OpenClawはTwitchユーザー（ボットアカウント）として接続し、チャンネル内でメッセージの送受信を行います。

## 必要なプラグイン

Twitchはプラグインとして提供されており、コアインストールにはバンドルされていません。CLI（npmレジストリ）経由でインストールします：

```bash
openclaw plugins install @openclaw/twitch
```

ローカルチェックアウト（gitリポジトリから実行する場合）：

```bash
openclaw plugins install ./extensions/twitch
```

詳細：[プラグイン](../tools/plugin.md)

## クイックセットアップ（初心者向け）

1.  ボット用の専用Twitchアカウントを作成します（または既存のアカウントを使用します）。
2.  認証情報を生成します：[Twitch Token Generator](https://twitchtokengenerator.com/)
    -   **Bot Token**を選択
    -   スコープ`chat:read`と`chat:write`が選択されていることを確認
    -   **Client ID**と**Access Token**をコピー
3.  TwitchユーザーIDを調べます：[https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/)
4.  トークンを設定します：
    -   環境変数：`OPENCLAW_TWITCH_ACCESS_TOKEN=...`（デフォルトアカウントのみ）
    -   または設定ファイル：`channels.twitch.accessToken`
    -   両方が設定されている場合、設定ファイルが優先されます（環境変数はデフォルトアカウントのみのフォールバック）。
5.  ゲートウェイを起動します。

**⚠️ 重要：** 未承認ユーザーによるボットのトリガーを防ぐため、アクセス制御（`allowFrom`または`allowedRoles`）を追加してください。`requireMention`はデフォルトで`true`です。最小限の設定：

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw", // ボットのTwitchアカウント
      accessToken: "oauth:abc123...", // OAuthアクセストークン（またはOPENCLAW_TWITCH_ACCESS_TOKEN環境変数を使用）
      clientId: "xyz789...", // Token GeneratorからのクライアントID
      channel: "vevisk", // 参加するTwitchチャンネル（必須）
      allowFrom: ["123456789"], // （推奨）あなたのTwitchユーザーIDのみ - https://www.streamweasels.com/tools/convert-twitch-username-to-user-id/ から取得
    },
  },
}
```

## 概要

-   ゲートウェイが所有するTwitchチャンネル。
-   決定論的なルーティング：返信は常にTwitchに戻ります。
-   各アカウントは独立したセッションキー`agent::twitch:`にマッピングされます。
-   `username`はボットのアカウント（認証する側）、`channel`は参加するチャットルームです。

## セットアップ（詳細）

### 認証情報の生成

[Twitch Token Generator](https://twitchtokengenerator.com/)を使用：

-   **Bot Token**を選択
-   スコープ`chat:read`と`chat:write`が選択されていることを確認
-   **Client ID**と**Access Token**をコピー

手動でのアプリ登録は不要です。トークンは数時間で期限切れになります。

### ボットの設定

**環境変数（デフォルトアカウントのみ）：**

```
OPENCLAW_TWITCH_ACCESS_TOKEN=oauth:abc123...
```

**または設定ファイル：**

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
    },
  },
}
```

環境変数と設定ファイルの両方が設定されている場合、設定ファイルが優先されます。

### アクセス制御（推奨）

```json
{
  channels: {
    twitch: {
      allowFrom: ["123456789"], // （推奨）あなたのTwitchユーザーIDのみ
    },
  },
}
```

厳格な許可リストには`allowFrom`を推奨します。ロールベースのアクセスが必要な場合は、代わりに`allowedRoles`を使用してください。**利用可能なロール：** `"moderator"`、`"owner"`、`"vip"`、`"subscriber"`、`"all"`。**なぜユーザーIDか？** ユーザー名は変更可能で、なりすましを許す可能性があります。ユーザーIDは永続的です。TwitchユーザーIDを調べる：[https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/](https://www.streamweasels.com/tools/convert-twitch-username-%20to-user-id/)（Twitchユーザー名をIDに変換）

## トークン更新（オプション）

[Twitch Token Generator](https://twitchtokengenerator.com/)からのトークンは自動更新できません - 期限切れになったら再生成してください。自動トークン更新のためには、[Twitch Developer Console](https://dev.twitch.tv/console)で独自のTwitchアプリケーションを作成し、設定に追加します：

```json
{
  channels: {
    twitch: {
      clientSecret: "your_client_secret",
      refreshToken: "your_refresh_token",
    },
  },
}
```

ボットは期限切れ前にトークンを自動更新し、更新イベントをログに記録します。

## マルチアカウントサポート

アカウントごとのトークンで`channels.twitch.accounts`を使用します。共有パターンについては[`gateway/configuration`](../gateway/configuration.md)を参照してください。例（1つのボットアカウントで2つのチャンネル）：

```json
{
  channels: {
    twitch: {
      accounts: {
        channel1: {
          username: "openclaw",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "vevisk",
        },
        channel2: {
          username: "openclaw",
          accessToken: "oauth:def456...",
          clientId: "uvw012...",
          channel: "secondchannel",
        },
      },
    },
  },
}
```

**注意：** 各アカウントには独自のトークンが必要です（チャンネルごとに1つのトークン）。

## アクセス制御

### ロールベースの制限

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator", "vip"],
        },
      },
    },
  },
}
```

### ユーザーIDによる許可リスト（最も安全）

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowFrom: ["123456789", "987654321"],
        },
      },
    },
  },
}
```

### ロールベースアクセス（代替案）

`allowFrom`は厳格な許可リストです。設定すると、それらのユーザーIDのみが許可されます。ロールベースのアクセスが必要な場合は、`allowFrom`を設定せず、代わりに`allowedRoles`を設定してください：

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

### @メンション要件の無効化

デフォルトでは、`requireMention`は`true`です。無効にしてすべてのメッセージに応答するには：

```json
{
  channels: {
    twitch: {
      accounts: {
        default: {
          requireMention: false,
        },
      },
    },
  },
}
```

## トラブルシューティング

まず、診断コマンドを実行します：

```bash
openclaw doctor
openclaw channels status --probe
```

### ボットがメッセージに応答しない

**アクセス制御を確認：** あなたのユーザーIDが`allowFrom`に含まれていることを確認するか、一時的に`allowFrom`を削除し、`allowedRoles: ["all"]`を設定してテストします。**ボットがチャンネルに参加しているか確認：** ボットは`channel`で指定されたチャンネルに参加している必要があります。

### トークンの問題

**「接続に失敗しました」または認証エラー：**

-   `accessToken`がOAuthアクセストークンの値であることを確認（通常は`oauth:`プレフィックスで始まる）
-   トークンに`chat:read`と`chat:write`スコープがあることを確認
-   トークン更新を使用している場合は、`clientSecret`と`refreshToken`が設定されていることを確認

### トークン更新が機能しない

**更新イベントのログを確認：**

```bash
Using env token source for mybot
Access token refreshed for user 123456 (expires in 14400s)
```

「token refresh disabled (no refresh token)」が表示される場合：

-   `clientSecret`が提供されていることを確認
-   `refreshToken`が提供されていることを確認

## 設定

**アカウント設定：**

-   `username` - ボットのユーザー名
-   `accessToken` - `chat:read`と`chat:write`を持つOAuthアクセストークン
-   `clientId` - TwitchクライアントID（Token Generatorまたは独自のアプリから）
-   `channel` - 参加するチャンネル（必須）
-   `enabled` - このアカウントを有効化（デフォルト：`true`）
-   `clientSecret` - オプション：自動トークン更新用
-   `refreshToken` - オプション：自動トークン更新用
-   `expiresIn` - トークンの有効期限（秒）
-   `obtainmentTimestamp` - トークン取得タイムスタンプ
-   `allowFrom` - ユーザーID許可リスト
-   `allowedRoles` - ロールベースアクセス制御（`"moderator" | "owner" | "vip" | "subscriber" | "all"`）
-   `requireMention` - @メンションを必須とする（デフォルト：`true`）

**プロバイダーオプション：**

-   `channels.twitch.enabled` - チャンネル起動の有効/無効
-   `channels.twitch.username` - ボットユーザー名（簡易単一アカウント設定）
-   `channels.twitch.accessToken` - OAuthアクセストークン（簡易単一アカウント設定）
-   `channels.twitch.clientId` - TwitchクライアントID（簡易単一アカウント設定）
-   `channels.twitch.channel` - 参加するチャンネル（簡易単一アカウント設定）
-   `channels.twitch.accounts.` - マルチアカウント設定（上記のすべてのアカウントフィールド）

完全な例：

```json
{
  channels: {
    twitch: {
      enabled: true,
      username: "openclaw",
      accessToken: "oauth:abc123...",
      clientId: "xyz789...",
      channel: "vevisk",
      clientSecret: "secret123...",
      refreshToken: "refresh456...",
      allowFrom: ["123456789"],
      allowedRoles: ["moderator", "vip"],
      accounts: {
        default: {
          username: "mybot",
          accessToken: "oauth:abc123...",
          clientId: "xyz789...",
          channel: "your_channel",
          enabled: true,
          clientSecret: "secret123...",
          refreshToken: "refresh456...",
          expiresIn: 14400,
          obtainmentTimestamp: 1706092800000,
          allowFrom: ["123456789", "987654321"],
          allowedRoles: ["moderator"],
        },
      },
    },
  },
}
```

## ツールアクション

エージェントは`twitch`アクションを呼び出せます：

-   `send` - チャンネルにメッセージを送信

例：

```json
{
  action: "twitch",
  params: {
    message: "Hello Twitch!",
    to: "#mychannel",
  },
}
```

## 安全性と運用

-   **トークンはパスワードのように扱う** - トークンをgitにコミットしない
-   **長時間実行するボットには自動トークン更新を使用**
-   アクセス制御にはユーザー名ではなく**ユーザーID許可リストを使用**
-   トークン更新イベントと接続ステータスの**ログを監視**
-   **トークンのスコープは最小限に** - `chat:read`と`chat:write`のみをリクエスト
-   **問題が解決しない場合：** 他のプロセスがセッションを所有していないことを確認した後、ゲートウェイを再起動

## 制限

-   **メッセージあたり500文字**（単語境界で自動分割）
-   マークダウンは分割前に除去
-   レート制限なし（Twitchの組み込みレート制限を使用）

[Tlon](./tlon.md)[WhatsApp](./whatsapp.md)