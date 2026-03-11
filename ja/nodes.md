

  メディアとデバイス

  
# ノード

**ノード** は、ゲートウェイの **WebSocket** (オペレーターと同じポート) に `role: "node"` で接続し、`node.invoke` を介してコマンドサーフェス (例: `canvas.*`, `camera.*`, `device.*`, `notifications.*`, `system.*`) を公開するコンパニオンデバイス (macOS/iOS/Android/ヘッドレス) です。プロトコルの詳細: [ゲートウェイプロトコル](./gateway/protocol.md)。レガシートランスポート: [ブリッジプロトコル](./gateway/bridge-protocol.md) (TCP JSONL; 現在のノードでは非推奨/削除)。macOS は **ノードモード** でも実行できます: メニューバーアプリがゲートウェイの WS サーバーに接続し、ローカルのキャンバス/カメラコマンドをノードとして公開します (そのため、この Mac に対して `openclaw nodes …` が機能します)。注意点:

-   ノードは **周辺機器** であり、ゲートウェイサービスを実行しません。
-   Telegram/WhatsApp などのメッセージは、ノードではなく **ゲートウェイ** に届きます。
-   トラブルシューティング手順書: [/nodes/troubleshooting](./nodes/troubleshooting.md)

## ペアリング + ステータス

**WS ノードはデバイスペアリングを使用します。** ノードは `connect` 中にデバイス ID を提示し、ゲートウェイは `role: node` のデバイスペアリングリクエストを作成します。デバイス CLI (または UI) で承認します。クイック CLI:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
```

注意点:

-   `nodes status` は、デバイスペアリングのロールに `node` が含まれている場合、ノードを **ペアリング済み** とマークします。
-   `node.pair.*` (CLI: `openclaw nodes pending/approve/reject`) は別のゲートウェイ所有のノードペアリングストアであり、WS `connect` ハンドシェイクをゲートし **ません**。

## リモートノードホスト (system.run)

ゲートウェイが1台のマシンで実行されていて、別のマシンでコマンドを実行したい場合に **ノードホスト** を使用します。モデルは依然として **ゲートウェイ** と通信し、ゲートウェイは `host=node` が選択されたときに `exec` 呼び出しを **ノードホスト** に転送します。

### 何がどこで実行されるか

-   **ゲートウェイホスト**: メッセージを受信し、モデルを実行し、ツール呼び出しをルーティングします。
-   **ノードホスト**: ノードマシン上で `system.run`/`system.which` を実行します。
-   **承認**: ノードホスト上で `~/.openclaw/exec-approvals.json` を介して強制されます。

### ノードホストの起動 (フォアグラウンド)

ノードマシン上で:

```bash
openclaw node run --host <gateway-host> --port 18789 --display-name "Build Node"
```

### SSH トンネルを介したリモートゲートウェイ (ループバックバインド)

ゲートウェイがループバックにバインドしている場合 (`gateway.bind=loopback`、ローカルモードでのデフォルト)、リモートノードホストは直接接続できません。SSH トンネルを作成し、ノードホストをトンネルのローカル側に向けます。例 (ノードホスト -> ゲートウェイホスト):

```bash
# ターミナル A (実行し続ける): ローカル 18790 -> ゲートウェイ 127.0.0.1:18789 を転送
ssh -N -L 18790:127.0.0.1:18789 user@gateway-host

# ターミナル B: ゲートウェイトークンをエクスポートし、トンネル経由で接続
export OPENCLAW_GATEWAY_TOKEN="<gateway-token>"
openclaw node run --host 127.0.0.1 --port 18790 --display-name "Build Node"
```

注意点:

-   トークンはゲートウェイホスト上のゲートウェイ設定 (`~/.openclaw/openclaw.json`) からの `gateway.auth.token` です。
-   `openclaw node run` は認証のために `OPENCLAW_GATEWAY_TOKEN` を読み取ります。

### ノードホストの起動 (サービス)

```bash
openclaw node install --host <gateway-host> --port 18789 --display-name "Build Node"
openclaw node restart
```

### ペアリング + 名前付け

ゲートウェイホスト上で:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw nodes status
```

名前付けオプション:

-   `openclaw node run` / `openclaw node install` での `--display-name` (ノード上の `~/.openclaw/node.json` に永続化)。
-   `openclaw nodes rename --node <id|name|ip> --name "Build Node"` (ゲートウェイによる上書き)。

### コマンドを許可リストに追加

実行承認は **ノードホストごと** です。ゲートウェイから許可リストエントリを追加します:

```bash
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/uname"
openclaw approvals allowlist add --node <id|name|ip> "/usr/bin/sw_vers"
```

承認はノードホスト上の `~/.openclaw/exec-approvals.json` に保存されます。

### exec をノードに向ける

デフォルトを設定 (ゲートウェイ設定):

```bash
openclaw config set tools.exec.host node
openclaw config set tools.exec.security allowlist
openclaw config set tools.exec.node "<id-or-name>"
```

またはセッションごと:

```bash
/exec host=node security=allowlist node=<id-or-name>
```

一度設定すると、`host=node` を持つ `exec` 呼び出しはすべてノードホスト上で実行されます (ノードの許可リスト/承認の対象となります)。関連項目:

-   [ノードホスト CLI](./cli/node.md)
-   [Exec ツール](./tools/exec.md)
-   [Exec 承認](./tools/exec-approvals.md)

## コマンドの呼び出し

低レベル (生の RPC):

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command canvas.eval --params '{"javaScript":"location.href"}'
```

一般的な「エージェントに MEDIA 添付ファイルを与える」ワークフロー向けに、より高レベルのヘルパーが存在します。

## スクリーンショット (キャンバススナップショット)

ノードがキャンバス (WebView) を表示している場合、`canvas.snapshot` は `{ format, base64 }` を返します。CLI ヘルパー (一時ファイルに書き込み、`MEDIA:` を出力):

```bash
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format png
openclaw nodes canvas snapshot --node <idOrNameOrIp> --format jpg --max-width 1200 --quality 0.9
```

### キャンバス制御

```bash
openclaw nodes canvas present --node <idOrNameOrIp> --target https://example.com
openclaw nodes canvas hide --node <idOrNameOrIp>
openclaw nodes canvas navigate https://example.com --node <idOrNameOrIp>
openclaw nodes canvas eval --node <idOrNameOrIp> --js "document.title"
```

注意点:

-   `canvas present` は URL またはローカルファイルパス (`--target`) を受け入れ、オプションで `--x/--y/--width/--height` で位置を指定できます。
-   `canvas eval` はインライン JS (`--js`) または位置引数を受け入れます。

### A2UI (キャンバス)

```bash
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --text "Hello"
openclaw nodes canvas a2ui push --node <idOrNameOrIp> --jsonl ./payload.jsonl
openclaw nodes canvas a2ui reset --node <idOrNameOrIp>
```

注意点:

-   A2UI v0.8 JSONL のみサポートされています (v0.9/createSurface は拒否されます)。

## 写真 + 動画 (ノードカメラ)

写真 (`jpg`):

```bash
openclaw nodes camera list --node <idOrNameOrIp>
openclaw nodes camera snap --node <idOrNameOrIp>            # デフォルト: 両方の向き (2 MEDIA 行)
openclaw nodes camera snap --node <idOrNameOrIp> --facing front
```

動画クリップ (`mp4`):

```bash
openclaw nodes camera clip --node <idOrNameOrIp> --duration 10s
openclaw nodes camera clip --node <idOrNameOrIp> --duration 3000 --no-audio
```

注意点:

-   `canvas.*` および `camera.*` のためには、ノードを **フォアグラウンド** にする必要があります (バックグラウンド呼び出しは `NODE_BACKGROUND_UNAVAILABLE` を返します)。
-   クリップの長さは、過大な base64 ペイロードを避けるために制限されます (現在 `<= 60s`)。
-   Android では可能な場合に `CAMERA`/`RECORD_AUDIO` 権限のプロンプトが表示されます。拒否された権限は `*_PERMISSION_REQUIRED` で失敗します。

## 画面録画 (ノード)

ノードは `screen.record` (mp4) を公開します。例:

```bash
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10
openclaw nodes screen record --node <idOrNameOrIp> --duration 10s --fps 10 --no-audio
```

注意点:

-   `screen.record` にはノードアプリがフォアグラウンドである必要があります。
-   Android では録画前にシステムの画面キャプチャプロンプトが表示されます。
-   画面録画は `<= 60s` に制限されます。
-   `--no-audio` はマイクキャプチャを無効にします (iOS/Android でサポート; macOS はシステムキャプチャオーディオを使用)。
-   複数の画面が利用可能な場合、`--screen ` を使用してディスプレイを選択します。

## 位置情報 (ノード)

設定で位置情報が有効になっている場合、ノードは `location.get` を公開します。CLI ヘルパー:

```bash
openclaw nodes location get --node <idOrNameOrIp>
openclaw nodes location get --node <idOrNameOrIp> --accuracy precise --max-age 15000 --location-timeout 10000
```

注意点:

-   位置情報は **デフォルトでオフ** です。
-   「常に」はシステム権限を必要とし、バックグラウンド取得はベストエフォートです。
-   応答には緯度/経度、精度 (メートル)、タイムスタンプが含まれます。

## SMS (Android ノード)

Android ノードは、ユーザーが **SMS** 権限を付与し、デバイスがテレフォニーをサポートしている場合に `sms.send` を公開できます。低レベル呼び出し:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command sms.send --params '{"to":"+15555550123","message":"Hello from OpenClaw"}'
```

注意点:

-   機能がアドバタイズされる前に、Android デバイスで権限プロンプトを受け入れる必要があります。
-   テレフォニーなしの Wi-Fi 専用デバイスは `sms.send` をアドバタイズしません。

## Android デバイス + 個人データコマンド

Android ノードは、対応する機能が有効になっている場合、追加のコマンドファミリーをアドバタイズできます。利用可能なファミリー:

-   `device.status`, `device.info`, `device.permissions`, `device.health`
-   `notifications.list`, `notifications.actions`
-   `photos.latest`
-   `contacts.search`, `contacts.add`
-   `calendar.events`, `calendar.add`
-   `motion.activity`, `motion.pedometer`
-   `app.update`

呼び出し例:

```bash
openclaw nodes invoke --node <idOrNameOrIp> --command device.status --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command notifications.list --params '{}'
openclaw nodes invoke --node <idOrNameOrIp> --command photos.latest --params '{"limit":1}'
```

注意点:

-   モーションコマンドは利用可能なセンサーによって機能が制限されます。
-   `app.update` はノードランタイムによって権限とポリシーで制限されます。

## システムコマンド (ノードホスト / mac ノード)

macOS ノードは `system.run`, `system.notify`, `system.execApprovals.get/set` を公開します。ヘッドレスノードホストは `system.run`, `system.which`, `system.execApprovals.get/set` を公開します。例:

```bash
openclaw nodes run --node <idOrNameOrIp> -- echo "Hello from mac node"
openclaw nodes notify --node <idOrNameOrIp> --title "Ping" --body "Gateway ready"
```

注意点:

-   `system.run` はペイロードに stdout/stderr/終了コードを返します。
-   `system.notify` は macOS アプリの通知権限状態を尊重します。
-   認識されないノードの `platform` / `deviceFamily` メタデータは、`system.run` と `system.which` を除外する保守的なデフォルト許可リストを使用します。未知のプラットフォームに対して意図的にそれらのコマンドが必要な場合は、`gateway.nodes.allowCommands` を介して明示的に追加してください。
-   `system.run` は `--cwd`, `--env KEY=VAL`, `--command-timeout`, `--needs-screen-recording` をサポートします。
-   シェルラッパー (`bash|sh|zsh ... -c/-lc`) の場合、リクエストスコープの `--env` 値は明示的な許可リスト (`TERM`, `LANG`, `LC_*`, `COLORTERM`, `NO_COLOR`, `FORCE_COLOR`) に削減されます。
-   許可リストモードでの常に許可する決定において、既知のディスパッチラッパー (`env`, `nice`, `nohup`, `stdbuf`, `timeout`) はラッパーパスの代わりに内部実行可能パスを保持します。アンラップが安全でない場合、許可リストエントリは自動的に保持されません。
-   許可リストモードの Windows ノードホストでは、`cmd.exe /c` 経由のシェルラッパーの実行には承認が必要です (許可リストエントリだけではラッパーフォームを自動許可しません)。
-   `system.notify` は `--priority <passive|active|timeSensitive>` と `--delivery <system|overlay|auto>` をサポートします。
-   ノードホストは `PATH` の上書きを無視し、危険な起動/シェルキー (`DYLD_*`, `LD_*`, `NODE_OPTIONS`, `PYTHON*`, `PERL*`, `RUBYOPT`, `SHELLOPTS`, `PS4`) を除去します。追加の PATH エントリが必要な場合は、`--env` 経由で `PATH` を渡す代わりに、ノードホストサービスの環境を設定するか、標準的な場所にツールをインストールしてください。
-   macOS ノードモードでは、`system.run` は macOS アプリ (設定 → 実行承認) の実行承認によって制限されます。許可リスト/完全な動作はヘッドレスノードホストと同じです。拒否されたプロンプトは `SYSTEM_RUN_DENIED` を返します。
-   ヘッドレスノードホストでは、`system.run` は実行承認 (`~/.openclaw/exec-approvals.json`) によって制限されます。

## Exec ノードバインディング

複数のノードが利用可能な場合、exec を特定のノードにバインドできます。これは `exec host=node` のデフォルトノードを設定します (エージェントごとに上書き可能)。グローバルデフォルト:

```bash
openclaw config set tools.exec.node "node-id-or-name"
```

エージェントごとの上書き:

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

任意のノードを許可するように解除:

```bash
openclaw config unset tools.exec.node
openclaw config unset agents.list[0].tools.exec.node
```

## 権限マップ

ノードは `node.list` / `node.describe` に権限名 (例: `screenRecording`, `accessibility`) をキーとし、ブール値 (`true` = 付与済み) を持つ `permissions` マップを含む場合があります。

## ヘッドレスノードホスト (クロスプラットフォーム)

OpenClaw は、ゲートウェイ WebSocket に接続し `system.run` / `system.which` を公開する **ヘッドレスノードホスト** (UI なし) を実行できます。これは Linux/Windows 上や、サーバーと一緒に最小限のノードを実行する場合に便利です。起動:

```bash
openclaw node run --host <gateway-host> --port 18789
```

注意点:

-   ペアリングは依然として必要です (ゲートウェイはデバイスペアリングプロンプトを表示します)。
-   ノードホストはそのノード ID、トークン、表示名、ゲートウェイ接続情報を `~/.openclaw/node.json` に保存します。
-   実行承認は `~/.openclaw/exec-approvals.json` を介してローカルで強制されます ( [実行承認](./tools/exec-approvals.md) を参照)。
-   macOS では、ヘッドレスノードホストはデフォルトでローカルで `system.run` を実行します。`OPENCLAW_NODE_EXEC_HOST=app` を設定して `system.run` をコンパニオンアプリの実行ホスト経由でルーティングします。`OPENCLAW_NODE_EXEC_FALLBACK=0` を追加してアプリホストを要求し、利用できない場合は閉じて失敗させます。
-   ゲートウェイ WS が TLS を使用する場合、`--tls` / `--tls-fingerprint` を追加します。

## Mac ノードモード

-   macOS メニューバーアプリは、ノードとしてゲートウェイ WS サーバーに接続します (そのため、この Mac に対して `openclaw nodes …` が機能します)。
-   リモートモードでは、アプリはゲートウェイポート用の SSH トンネルを開き、`localhost` に接続します。

[認証モニタリング](./automation/auth-monitoring.md)[ノードトラブルシューティング](./nodes/troubleshooting.md)