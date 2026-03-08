

  ヘルプ

  
# トラブルシューティング

時間が2分しかない場合は、このページをトリアージの入り口として使用してください。

## 最初の60秒

この正確なラダーを順番に実行してください:

```bash
openclaw status
openclaw status --all
openclaw gateway probe
openclaw gateway status
openclaw doctor
openclaw channels status --probe
openclaw logs --follow
```

1行での良好な出力:

-   `openclaw status` → 設定されたチャネルが表示され、明らかな認証エラーがない。
-   `openclaw status --all` → 完全なレポートが存在し、共有可能である。
-   `openclaw gateway probe` → 期待されるゲートウェイターゲットに到達可能。
-   `openclaw gateway status` → `Runtime: running` および `RPC probe: ok`。
-   `openclaw doctor` → ブロッキングする設定/サービスエラーがない。
-   `openclaw channels status --probe` → チャネルが `connected` または `ready` と報告する。
-   `openclaw logs --follow` → 安定したアクティビティ、繰り返し発生する致命的なエラーがない。

## Anthropic 長文コンテキスト 429

`HTTP 429: rate_limit_error: Extra usage is required for long context requests` が表示される場合、[/gateway/troubleshooting#anthropic-429-extra-usage-required-for-long-context](../gateway/troubleshooting.md#anthropic-429-extra-usage-required-for-long-context) に移動してください。

## プラグインインストールが openclaw extensions の欠如で失敗する

インストールが `package.json missing openclaw.extensions` で失敗する場合、プラグインパッケージがOpenClawが受け入れなくなった古い形状を使用しています。プラグインパッケージ内で修正してください:

1.  `package.json` に `openclaw.extensions` を追加する。
2.  エントリをビルド済みランタイムファイル（通常は `./dist/index.js`）に向ける。
3.  プラグインを再公開し、`openclaw plugins install <npm-spec>` を再度実行する。

例:

```json
{
  "name": "@openclaw/my-plugin",
  "version": "1.2.3",
  "openclaw": {
    "extensions": ["./dist/index.js"]
  }
}
```

参照: [/tools/plugin#distribution-npm](../tools/plugin.md#distribution-npm)

## 決定木

```bash
openclaw status
openclaw gateway status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
```

良好な出力は以下のようになります:

-   `Runtime: running`
-   `RPC probe: ok`
-   あなたのチャネルが `channels status --probe` で connected/ready と表示される
-   送信者が承認済み（またはDMポリシーが open/allowlist）である

一般的なログの特徴:

-   `drop guild message (mention required` → メンションゲーティングがDiscord内のメッセージをブロックした。
-   `pairing request` → 送信者が未承認で、DMペアリング承認を待機中。
-   チャネルログ内の `blocked` / `allowlist` → 送信者、ルーム、またはグループがフィルタリングされた。

詳細ページ:

-   [/gateway/troubleshooting#no-replies](../gateway/troubleshooting.md#no-replies)
-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

良好な出力は以下のようになります:

-   `Dashboard: http://...` が `openclaw gateway status` に表示される
-   `RPC probe: ok`
-   ログに認証ループがない

一般的なログの特徴:

-   `device identity required` → HTTP/非セキュアコンテキストはデバイス認証を完了できない。
-   `unauthorized` / 再接続ループ → 間違ったトークン/パスワードまたは認証モードの不一致。
-   `gateway connect failed:` → UIが間違ったURL/ポートをターゲットにしている、またはゲートウェイに到達できない。

詳細ページ:

-   [/gateway/troubleshooting#dashboard-control-ui-connectivity](../gateway/troubleshooting.md#dashboard-control-ui-connectivity)
-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](../gateway/authentication.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

良好な出力は以下のようになります:

-   `Service: ... (loaded)`
-   `Runtime: running`
-   `RPC probe: ok`

一般的なログの特徴:

-   `Gateway start blocked: set gateway.mode=local` → ゲートウェイモードが未設定/remote。
-   `refusing to bind gateway ... without auth` → トークン/パスワードなしでの非ループバックバインド。
-   `another gateway instance is already listening` または `EADDRINUSE` → ポートが既に使用中。

詳細ページ:

-   [/gateway/troubleshooting#gateway-service-not-running](../gateway/troubleshooting.md#gateway-service-not-running)
-   [/gateway/background-process](../gateway/background-process.md)
-   [/gateway/configuration](../gateway/configuration.md)

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

良好な出力は以下のようになります:

-   チャネルトランスポートが接続されている。
-   ペアリング/許可リストチェックが通過する。
-   必要な場所でメンションが検出される。

一般的なログの特徴:

-   `mention required` → グループメンションゲーティングが処理をブロックした。
-   `pairing` / `pending` → DM送信者がまだ承認されていない。
-   `not_in_channel`, `missing_scope`, `Forbidden`, `401/403` → チャネル権限トークンの問題。

詳細ページ:

-   [/gateway/troubleshooting#channel-connected-messages-not-flowing](../gateway/troubleshooting.md#channel-connected-messages-not-flowing)
-   [/channels/troubleshooting](../channels/troubleshooting.md)

```bash
openclaw status
openclaw gateway status
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw logs --follow
```

良好な出力は以下のようになります:

-   `cron.status` が有効で次のウェイクを表示する。
-   `cron runs` が最近の `ok` エントリを表示する。
-   ハートビートが有効で、アクティブ時間外ではない。

一般的なログの特徴:

-   `cron: scheduler disabled; jobs will not run automatically` → cronが無効。
-   `heartbeat skipped` と `reason=quiet-hours` → 設定されたアクティブ時間外。
-   `requests-in-flight` → メインレーンがビジー; ハートビートウェイクが延期された。
-   `unknown accountId` → ハートビート配信ターゲットアカウントが存在しない。

詳細ページ:

-   [/gateway/troubleshooting#cron-and-heartbeat-delivery](../gateway/troubleshooting.md#cron-and-heartbeat-delivery)
-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/gateway/heartbeat](../gateway/heartbeat.md)

```bash
openclaw status
openclaw gateway status
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw logs --follow
```

良好な出力は以下のようになります:

-   ノードが接続済みとしてリストされ、ロール `node` に対してペアリングされている。
-   呼び出しているコマンドに対して機能が存在する。
-   ツールに対して許可状態が付与されている。

一般的なログの特徴:

-   `NODE_BACKGROUND_UNAVAILABLE` → ノードアプリをフォアグラウンドに持ってくる。
-   `*_PERMISSION_REQUIRED` → OS権限が拒否/欠如した。
-   `SYSTEM_RUN_DENIED: approval required` → exec承認が保留中。
-   `SYSTEM_RUN_DENIED: allowlist miss` → コマンドがexec許可リストにない。

詳細ページ:

-   [/gateway/troubleshooting#node-paired-tool-fails](../gateway/troubleshooting.md#node-paired-tool-fails)
-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

```bash
openclaw status
openclaw gateway status
openclaw browser status
openclaw logs --follow
openclaw doctor
```

良好な出力は以下のようになります:

-   ブラウザステータスが `running: true` と選択されたブラウザ/プロファイルを表示する。
-   `openclaw` プロファイルが起動する、または `chrome` リレーに接続されたタブがある。

一般的なログの特徴:

-   `Failed to start Chrome CDP on port` → ローカルブラウザ起動に失敗。
-   `browser.executablePath not found` → 設定されたバイナリパスが間違っている。
-   `Chrome extension relay is running, but no tab is connected` → 拡張機能が接続されていない。
-   `Browser attachOnly is enabled ... not reachable` → アタッチ専用プロファイルにライブCDPターゲットがない。

詳細ページ:

-   [/gateway/troubleshooting#browser-tool-fails](../gateway/troubleshooting.md#browser-tool-fails)
-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)

[ヘルプ](../help.md)[よくある質問](./faq.md)