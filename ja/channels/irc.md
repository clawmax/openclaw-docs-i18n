

  メッセージングプラットフォーム

  
# IRC

OpenClawを従来のチャンネル (`#room`) やダイレクトメッセージで使用したい場合はIRCを使用します。IRCは拡張プラグインとして同梱されていますが、メイン設定の `channels.irc` で構成されます。

## クイックスタート

1.  `~/.openclaw/openclaw.json` でIRC設定を有効にします。
2.  少なくとも以下を設定します:

```json
{
  "channels": {
    "irc": {
      "enabled": true,
      "host": "irc.libera.chat",
      "port": 6697,
      "tls": true,
      "nick": "openclaw-bot",
      "channels": ["#openclaw"]
    }
  }
}
```

3.  ゲートウェイを起動/再起動します:

```bash
openclaw gateway run
```

## セキュリティのデフォルト

-   `channels.irc.dmPolicy` のデフォルトは `"pairing"` です。
-   `channels.irc.groupPolicy` のデフォルトは `"allowlist"` です。
-   `groupPolicy="allowlist"` の場合、許可されたチャンネルを定義するために `channels.irc.groups` を設定します。
-   意図的に平文転送を受け入れる場合を除き、TLS (`channels.irc.tls=true`) を使用してください。

## アクセス制御

IRCチャンネルには2つの独立した「ゲート」があります:

1.  **チャンネルアクセス** (`groupPolicy` + `groups`): ボットがチャンネルからのメッセージを全く受け入れるかどうか。
2.  **送信者アクセス** (`groupAllowFrom` / チャンネルごとの `groups["#channel"].allowFrom`): そのチャンネル内でボットをトリガーすることを許可されているのは誰か。

設定キー:

-   DM許可リスト (DM送信者アクセス): `channels.irc.allowFrom`
-   グループ送信者許可リスト (チャンネル送信者アクセス): `channels.irc.groupAllowFrom`
-   チャンネルごとの制御 (チャンネル + 送信者 + メンションルール): `channels.irc.groups["#channel"]`
-   `channels.irc.groupPolicy="open"` は未設定のチャンネルを許可します (**デフォルトでは依然としてメンションゲーティングされます**)

許可リストのエントリは安定した送信者識別子 (`nick!user@host`) を使用する必要があります。単なるニックネームのマッチングは変更可能であり、`channels.irc.dangerouslyAllowNameMatching: true` が設定されている場合にのみ有効になります。

### よくある落とし穴: allowFrom はDM用であり、チャンネル用ではありません

以下のようなログが表示される場合:

-   `irc: drop group sender alice!ident@host (policy=allowlist)`

…これは、送信者が**グループ/チャンネル**メッセージに対して許可されていなかったことを意味します。以下のいずれかで修正します:

-   `channels.irc.groupAllowFrom` を設定する (すべてのチャンネルでグローバル)、または
-   チャンネルごとの送信者許可リストを設定する: `channels.irc.groups["#channel"].allowFrom`

例 (`#tuirc-dev` 内の誰でもボットと話せるようにする):

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": { allowFrom: ["*"] },
      },
    },
  },
}
```

## 返信トリガー (メンション)

チャンネルが許可されていても ( `groupPolicy` + `groups` 経由で)、送信者が許可されていても、OpenClawはグループコンテキストではデフォルトで**メンションゲーティング**を行います。つまり、メッセージにボットに一致するメンションパターンが含まれていない限り、`drop channel … (missing-mention)` のようなログが表示される可能性があります。ボットがIRCチャンネルで**メンションを必要とせずに**返信するようにするには、そのチャンネルのメンションゲーティングを無効にします:

```json
{
  channels: {
    irc: {
      groupPolicy: "allowlist",
      groups: {
        "#tuirc-dev": {
          requireMention: false,
          allowFrom: ["*"],
        },
      },
    },
  },
}
```

または、**すべての**IRCチャンネルを許可し (チャンネルごとの許可リストなし)、それでもメンションなしで返信できるようにするには:

```json
{
  channels: {
    irc: {
      groupPolicy: "open",
      groups: {
        "*": { requireMention: false, allowFrom: ["*"] },
      },
    },
  },
}
```

## セキュリティ上の注意 (公開チャンネルでの推奨事項)

公開チャンネルで `allowFrom: ["*"]` を許可すると、誰でもボットにプロンプトを送信できます。リスクを軽減するには、そのチャンネルのツールを制限してください。

### チャンネル内の全員に同じツールを適用

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          tools: {
            deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
          },
        },
      },
    },
  },
}
```

### 送信者ごとに異なるツールを適用 (所有者により多くの権限)

`toolsBySender` を使用して、`"*"` にはより厳格なポリシーを、あなたのニックネームにはより緩やかなポリシーを適用します:

```json
{
  channels: {
    irc: {
      groups: {
        "#tuirc-dev": {
          allowFrom: ["*"],
          toolsBySender: {
            "*": {
              deny: ["group:runtime", "group:fs", "gateway", "nodes", "cron", "browser"],
            },
            "id:eigen": {
              deny: ["gateway", "nodes", "cron"],
            },
          },
        },
      },
    },
  },
}
```

注意点:

-   `toolsBySender` のキーは、IRC送信者識別子の値に `id:` を使用する必要があります: `id:eigen` または、より強力なマッチングの場合は `id:eigen!~eigen@174.127.248.171`。
-   レガシーな接頭辞なしのキーも受け入れられ、`id:` としてのみマッチングされます。
-   最初に一致した送信者ポリシーが優先されます; `"*"` はワイルドカードのフォールバックです。

グループアクセスとメンションゲーティング (およびそれらの相互作用) の詳細については、以下を参照してください: [/channels/groups](./groups.md)。

## NickServ

接続後にNickServで認証するには:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "enabled": true,
        "service": "NickServ",
        "password": "your-nickserv-password"
      }
    }
  }
}
```

接続時のオプションの1回限りの登録:

```json
{
  "channels": {
    "irc": {
      "nickserv": {
        "register": true,
        "registerEmail": "bot@example.com"
      }
    }
  }
}
```

ニックネームが登録された後は、繰り返しのREGISTER試行を避けるために `register` を無効にしてください。

## 環境変数

デフォルトアカウントは以下をサポートします:

-   `IRC_HOST`
-   `IRC_PORT`
-   `IRC_TLS`
-   `IRC_NICK`
-   `IRC_USERNAME`
-   `IRC_REALNAME`
-   `IRC_PASSWORD`
-   `IRC_CHANNELS` (カンマ区切り)
-   `IRC_NICKSERV_PASSWORD`
-   `IRC_NICKSERV_REGISTER_EMAIL`

## トラブルシューティング

-   ボットが接続するがチャンネルで返信しない場合、`channels.irc.groups` **および**メンションゲーティングがメッセージをドロップしているかどうか (`missing-mention`) を確認してください。ピングなしで返信させたい場合は、そのチャンネルに `requireMention:false` を設定します。
-   ログインに失敗する場合は、ニックネームの可用性とサーバーパスワードを確認してください。
-   カスタムネットワークでTLSが失敗する場合は、ホスト/ポートと証明書の設定を確認してください。

[iMessage](./imessage.md)[LINE](./line.md)