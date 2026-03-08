

  設定と運用

  
# トラブルシューティング

このページは詳細なランブックです。最初に迅速なトリアージフローが必要な場合は、[/help/troubleshooting](../help/troubleshooting.md) から始めてください。

## コマンドラダー

最初に、この順序で実行してください：

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

期待される正常なシグナル：

-   `openclaw gateway status` が `Runtime: running` と `RPC probe: ok` を示す。
-   `openclaw doctor` がブロックする設定/サービスの問題を報告しない。
-   `openclaw channels status --probe` が接続済み/準備完了のチャネルを示す。

## Anthropic 429 長いコンテキストには追加使用量が必要

ログ/エラーに以下が含まれる場合に使用します：`HTTP 429: rate_limit_error: Extra usage is required for long context requests`。

```bash
openclaw logs --follow
openclaw models status
openclaw config get agents.defaults.models
```

以下を確認してください：

-   選択された Anthropic Opus/Sonnet モデルに `params.context1m: true` が設定されている。
-   現在の Anthropic 認証情報が長いコンテキスト使用の対象ではない。
-   1M ベータパスを必要とする長いセッション/モデル実行でのみリクエストが失敗する。

修正オプション：

1.  通常のコンテキストウィンドウにフォールバックするために、そのモデルの `context1m` を無効にする。
2.  課金対象の Anthropic API キーを使用するか、サブスクリプションアカウントで Anthropic Extra Usage を有効にする。
3.  Anthropic の長いコンテキストリクエストが拒否された場合に実行が継続するように、フォールバックモデルを設定する。

関連：

-   [/providers/anthropic](../providers/anthropic.md)
-   [/reference/token-use](../reference/token-use.md)
-   [/help/faq#why-am-i-seeing-http-429-ratelimiterror-from-anthropic](../help/faq.md#why-am-i-seeing-http-429-ratelimiterror-from-anthropic)

## 応答がない

チャネルが起動しているが何も応答しない場合、何かを再接続する前に、ルーティングとポリシーを確認してください。

```bash
openclaw status
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw config get channels
openclaw logs --follow
```

以下を確認してください：

-   DM 送信者のペアリングが保留中。
-   グループメンションゲーティング (`requireMention`, `mentionPatterns`)。
-   チャネル/グループ許可リストの不一致。

一般的な兆候：

-   `drop guild message (mention required` → メンションされるまでグループメッセージは無視される。
-   `pairing request` → 送信者は承認が必要。
-   `blocked` / `allowlist` → 送信者/チャネルがポリシーでフィルタリングされた。

関連：

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/pairing](../channels/pairing.md)
-   [/channels/groups](../channels/groups.md)

## ダッシュボード制御 UI 接続性

ダッシュボード/制御 UI が接続できない場合、URL、認証モード、およびセキュアコンテキストの前提条件を検証してください。

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --json
```

以下を確認してください：

-   正しいプローブ URL とダッシュボード URL。
-   クライアントとゲートウェイ間の認証モード/トークンの不一致。
-   デバイス識別が必要な場所での HTTP 使用。

一般的な兆候：

-   `device identity required` → 非セキュアコンテキストまたはデバイス認証が欠如。
-   `device nonce required` / `device nonce mismatch` → クライアントがチャレンジベースのデバイス認証フロー (`connect.challenge` + `device.nonce`) を完了していない。
-   `device signature invalid` / `device signature expired` → クライアントが現在のハンドシェイクに対して間違ったペイロード（または古いタイムスタンプ）に署名した。
-   `unauthorized` / 再接続ループ → トークン/パスワードの不一致。
-   `gateway connect failed:` → 間違ったホスト/ポート/URL ターゲット。

デバイス認証 v2 移行チェック：

```bash
openclaw --version
openclaw doctor
openclaw gateway status
```

ログに nonce/署名エラーが表示される場合、接続クライアントを更新し、以下を確認してください：

1.  `connect.challenge` を待機する
2.  チャレンジに紐づくペイロードに署名する
3.  同じチャレンジ nonce で `connect.params.device.nonce` を送信する

関連：

-   [/web/control-ui](../web/control-ui.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/remote](./remote.md)

## Gateway サービスが実行されていない

サービスがインストールされているが、プロセスが起動したままにならない場合に使用します。

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
openclaw doctor
openclaw gateway status --deep
```

以下を確認してください：

-   終了のヒントと共に `Runtime: stopped` が表示される。
-   サービス設定の不一致 (`Config (cli)` 対 `Config (service)`)。
-   ポート/リスナーの競合。

一般的な兆候：

-   `Gateway start blocked: set gateway.mode=local` → ローカルゲートウェイモードが有効になっていない。修正：設定で `gateway.mode="local"` を設定する（または `openclaw configure` を実行）。専用の `openclaw` ユーザーを使用して Podman 経由で OpenClaw を実行している場合、設定は `~openclaw/.openclaw/openclaw.json` にあります。
-   `refusing to bind gateway ... without auth` → トークン/パスワードなしでの非ループバックバインド。
-   `another gateway instance is already listening` / `EADDRINUSE` → ポート競合。

関連：

-   [/gateway/background-process](./background-process.md)
-   [/gateway/configuration](./configuration.md)
-   [/gateway/doctor](./doctor.md)

## チャネル接続済みだがメッセージが流れない

チャネル状態が接続済みだがメッセージフローが停止している場合、ポリシー、権限、およびチャネル固有の配信ルールに焦点を当ててください。

```bash
openclaw channels status --probe
openclaw pairing list --channel <channel> [--account <id>]
openclaw status --deep
openclaw logs --follow
openclaw config get channels
```

以下を確認してください：

-   DM ポリシー (`pairing`, `allowlist`, `open`, `disabled`)。
-   グループ許可リストとメンション要件。
-   チャネル API 権限/スコープの欠如。

一般的な兆候：

-   `mention required` → グループメンションポリシーによりメッセージが無視された。
-   `pairing` / 承認保留中のトレース → 送信者が承認されていない。
-   `missing_scope`, `not_in_channel`, `Forbidden`, `401/403` → チャネル認証/権限の問題。

関連：

-   [/channels/troubleshooting](../channels/troubleshooting.md)
-   [/channels/whatsapp](../channels/whatsapp.md)
-   [/channels/telegram](../channels/telegram.md)
-   [/channels/discord](../channels/discord.md)

## Cron とハートビート配信

cron またはハートビートが実行されなかった、または配信されなかった場合、まずスケジューラの状態を確認し、次に配信ターゲットを確認してください。

```bash
openclaw cron status
openclaw cron list
openclaw cron runs --id <jobId> --limit 20
openclaw system heartbeat last
openclaw logs --follow
```

以下を確認してください：

-   Cron が有効で、次のウェイクが存在する。
-   ジョブ実行履歴のステータス (`ok`, `skipped`, `error`)。
-   ハートビートスキップ理由 (`quiet-hours`, `requests-in-flight`, `alerts-disabled`)。

一般的な兆候：

-   `cron: scheduler disabled; jobs will not run automatically` → cron が無効。
-   `cron: timer tick failed` → スケジューラティックが失敗；ファイル/ログ/ランタイムエラーを確認。
-   `heartbeat skipped` で `reason=quiet-hours` → アクティブ時間ウィンドウ外。
-   `heartbeat: unknown accountId` → ハートビート配信ターゲットのアカウント ID が無効。
-   `heartbeat skipped` で `reason=dm-blocked` → ハートビートターゲットが DM スタイルの宛先に解決されたが、`agents.defaults.heartbeat.directPolicy`（またはエージェントごとのオーバーライド）が `block` に設定されている。

関連：

-   [/automation/troubleshooting](../automation/troubleshooting.md)
-   [/automation/cron-jobs](../automation/cron-jobs.md)
-   [/gateway/heartbeat](./heartbeat.md)

## ノードペアリング済みツールが失敗する

ノードがペアリングされているがツールが失敗する場合、フォアグラウンド、権限、および承認状態を分離してください。

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
openclaw status
```

以下を確認してください：

-   ノードがオンラインで、期待される機能を持つ。
-   カメラ/マイク/位置情報/画面に対する OS 権限付与。
-   実行承認と許可リストの状態。

一般的な兆候：

-   `NODE_BACKGROUND_UNAVAILABLE` → ノードアプリはフォアグラウンドである必要がある。
-   `*_PERMISSION_REQUIRED` / `LOCATION_PERMISSION_REQUIRED` → OS 権限が欠如。
-   `SYSTEM_RUN_DENIED: approval required` → 実行承認が保留中。
-   `SYSTEM_RUN_DENIED: allowlist miss` → コマンドが許可リストによってブロックされた。

関連：

-   [/nodes/troubleshooting](../nodes/troubleshooting.md)
-   [/nodes/index](../nodes/index.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)

## ブラウザーツールが失敗する

ゲートウェイ自体は正常だが、ブラウザーツールのアクションが失敗する場合に使用します。

```bash
openclaw browser status
openclaw browser start --browser-profile openclaw
openclaw browser profiles
openclaw logs --follow
openclaw doctor
```

以下を確認してください：

-   有効なブラウザ実行可能ファイルパス。
-   CDP プロファイルの到達可能性。
-   `profile="chrome"` のための拡張機能リレータブの接続。

一般的な兆候：

-   `Failed to start Chrome CDP on port` → ブラウザプロセスの起動に失敗。
-   `browser.executablePath not found` → 設定されたパスが無効。
-   `Chrome extension relay is running, but no tab is connected` → 拡張機能リレーが接続されていない。
-   `Browser attachOnly is enabled ... not reachable` → アタッチ専用プロファイルに到達可能なターゲットがない。

関連：

-   [/tools/browser-linux-troubleshooting](../tools/browser-linux-troubleshooting.md)
-   [/tools/chrome-extension](../tools/chrome-extension.md)
-   [/tools/browser](../tools/browser.md)

## アップグレード後に何かが突然壊れた場合

アップグレード後の障害のほとんどは、設定のずれや、現在適用されているより厳格なデフォルト値によるものです。

### 1) 認証と URL オーバーライドの動作が変更された

```bash
openclaw gateway status
openclaw config get gateway.mode
openclaw config get gateway.remote.url
openclaw config get gateway.auth.mode
```

確認すべきこと：

-   `gateway.mode=remote` の場合、CLI 呼び出しはリモートをターゲットにしている可能性があり、ローカルサービスは正常である。
-   明示的な `--url` 呼び出しは、保存された認証情報にフォールバックしない。

一般的な兆候：

-   `gateway connect failed:` → 間違った URL ターゲット。
-   `unauthorized` → エンドポイントは到達可能だが認証が間違っている。

### 2) バインドと認証のガードレールがより厳格になった

```bash
openclaw config get gateway.bind
openclaw config get gateway.auth.token
openclaw gateway status
openclaw logs --follow
```

確認すべきこと：

-   非ループバックバインド (`lan`, `tailnet`, `custom`) には認証設定が必要。
-   `gateway.token` などの古いキーは `gateway.auth.token` を置き換えない。

一般的な兆候：

-   `refusing to bind gateway ... without auth` → バインドと認証の不一致。
-   ランタイムが実行中に `RPC probe: failed` → ゲートウェイは生きているが、現在の認証/URL ではアクセスできない。

### 3) ペアリングとデバイス識別状態が変更された

```bash
openclaw devices list
openclaw pairing list --channel <channel> [--account <id>]
openclaw logs --follow
openclaw doctor
```

確認すべきこと：

-   ダッシュボード/ノードの保留中のデバイス承認。
-   ポリシーまたは識別情報の変更後の保留中の DM ペアリング承認。

一般的な兆候：

-   `device identity required` → デバイス認証が満たされていない。
-   `pairing required` → 送信者/デバイスは承認される必要がある。

チェック後もサービス設定とランタイムが一致しない場合は、同じプロファイル/状態ディレクトリからサービスメタデータを再インストールしてください：

```bash
openclaw gateway install --force
openclaw gateway restart
```

関連：

-   [/gateway/pairing](./pairing.md)
-   [/gateway/authentication](./authentication.md)
-   [/gateway/background-process](./background-process.md)

[複数のゲートウェイ](./multiple-gateways.md)[セキュリティ](./security.md)