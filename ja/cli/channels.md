

  CLIコマンド

  
# channels

ゲートウェイ上のチャットチャネルアカウントとそのランタイムステータスを管理します。関連ドキュメント:

-   チャネルガイド: [Channels](../channels/index.md)
-   ゲートウェイ設定: [Configuration](../gateway/configuration.md)

## よく使うコマンド

```bash
openclaw channels list
openclaw channels status
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels logs --channel all
```

## アカウントの追加 / 削除

```bash
openclaw channels add --channel telegram --token <bot-token>
openclaw channels remove --channel telegram --delete
```

ヒント: `openclaw channels add --help` はチャネルごとのフラグ（トークン、アプリトークン、signal-cliパスなど）を表示します。フラグなしで `openclaw channels add` を実行すると、インタラクティブウィザードが以下のように促すことがあります:

-   選択したチャネルごとのアカウントID
-   それらのアカウントのオプションの表示名
-   `設定したチャネルアカウントをエージェントに今すぐバインドしますか？`

今すぐバインドを確認すると、ウィザードはどのエージェントが各設定済みチャネルアカウントを所有すべきかを尋ね、アカウントスコープのルーティングバインディングを書き込みます。同じルーティングルールは後から `openclaw agents bindings`、`openclaw agents bind`、`openclaw agents unbind` で管理することもできます（[agents](./agents.md) を参照）。まだ単一アカウントのトップレベル設定（`channels..accounts` エントリがない）を使用しているチャネルに非デフォルトアカウントを追加する場合、OpenClawはアカウントスコープの単一アカウントトップレベル値を `channels..accounts.default` に移動し、新しいアカウントを書き込みます。これにより、マルチアカウント形式への移行中に元のアカウントの動作が維持されます。ルーティング動作は一貫しています:

-   既存のチャネルのみのバインディング（`accountId` なし）は、デフォルトアカウントに引き続きマッチします。
-   `channels add` は非インタラクティブモードではバインディングを自動生成または書き換えません。
-   インタラクティブセットアップでは、オプションでアカウントスコープのバインディングを追加できます。

設定がすでに混合状態（名前付きアカウントが存在し、`default` がなく、トップレベルの単一アカウント値がまだ設定されている）だった場合は、`openclaw doctor --fix` を実行してアカウントスコープの値を `accounts.default` に移動してください。

## ログイン / ログアウト (インタラクティブ)

```bash
openclaw channels login --channel whatsapp
openclaw channels logout --channel whatsapp
```

## トラブルシューティング

-   広範な調査には `openclaw status --deep` を実行します。
-   ガイド付き修正には `openclaw doctor` を使用します。
-   `openclaw channels list` が `Claude: HTTP 403 ... user:profile` を出力する → 使用状況スナップショットに `user:profile` スコープが必要です。`--no-usage` を使用するか、claude.aiセッションキー（`CLAUDE_WEB_SESSION_KEY` / `CLAUDE_WEB_COOKIE`）を提供するか、Claude Code CLI経由で再認証してください。
-   `openclaw channels status` は、ゲートウェイに到達できない場合、設定のみのサマリーにフォールバックします。サポートされているチャネル認証情報がSecretRef経由で設定されているが、現在のコマンドパスで利用できない場合、そのアカウントは「未設定」として表示するのではなく、設定済みで劣化した注記付きで報告します。

## 機能プローブ

プロバイダーの機能ヒント（利用可能な場合はインテント/スコープ）と静的な機能サポートを取得します:

```bash
openclaw channels capabilities
openclaw channels capabilities --channel discord --target channel:123
```

注記:

-   `--channel` はオプションです。省略すると、すべてのチャネル（拡張機能を含む）がリストされます。
-   `--target` は `channel:` または生の数値チャネルIDを受け入れ、Discordにのみ適用されます。
-   プローブはプロバイダー固有です: Discordインテント + オプションのチャネル権限; Slackボット + ユーザースコープ; Telegramボットフラグ + ウェブフック; Signalデーモンバージョン; MS Teamsアプリトークン + Graphロール/スコープ（既知の場合は注記付き）。プローブがないチャネルは `Probe: unavailable` と報告します。

## 名前からIDへの解決

プロバイダーのディレクトリを使用して、チャネル/ユーザー名をIDに解決します:

```bash
openclaw channels resolve --channel slack "#general" "@jane"
openclaw channels resolve --channel discord "My Server/#support" "@someone"
openclaw channels resolve --channel matrix "Project Room"
```

注記:

-   ターゲットタイプを強制するには `--kind user|group|auto` を使用します。
-   解決は、複数のエントリが同じ名前を共有している場合、アクティブなマッチを優先します。
-   `channels resolve` は読み取り専用です。選択したアカウントがSecretRef経由で設定されているが、その認証情報が現在のコマンドパスで利用できない場合、コマンドは実行全体を中止するのではなく、注記付きの劣化した未解決結果を返します。

[browser](./browser.md)[clawbot](./clawbot.md)