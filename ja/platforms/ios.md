

  プラットフォーム概要

  
# iOSアプリ

利用状況: 内部プレビュー。iOSアプリはまだ一般公開されていません。

## 機能

-   WebSocket経由（LANまたはテイルネット）でGatewayに接続します。
-   ノード機能を公開します: Canvas、画面スナップショット、カメラキャプチャ、位置情報、トークモード、ボイスウェイク。
-   `node.invoke`コマンドを受信し、ノードステータスイベントを報告します。

## 要件

-   別のデバイス（macOS、Linux、またはWSL2経由のWindows）で動作するGateway。
-   ネットワークパス:
    -   同じLAN（Bonjour経由）、**または**
    -   テイルネット（ユニキャストDNS-SD経由、例: `openclaw.internal.`ドメイン）、**または**
    -   手動ホスト/ポート（フォールバック）。

## クイックスタート（ペアリング + 接続）

1.  Gatewayを起動します:

```bash
openclaw gateway --port 18789
```

2.  iOSアプリで、設定を開き、検出されたゲートウェイを選択します（または「手動ホスト」を有効にしてホスト/ポートを入力します）。
3.  ゲートウェイホストでペアリングリクエストを承認します:

```bash
openclaw devices list
openclaw devices approve <requestId>
```

4.  接続を確認します:

```bash
openclaw nodes status
openclaw gateway call node.list --params "{}"
```

## ディスカバリーパス

### Bonjour（LAN）

Gatewayは`local.`上で`_openclaw-gw._tcp`をアドバタイズします。iOSアプリはこれらを自動的にリスト表示します。

### テイルネット（クロスネットワーク）

mDNSがブロックされている場合は、ユニキャストDNS-SDゾーン（ドメインを選択、例: `openclaw.internal.`）とTailscaleスプリットDNSを使用します。詳細は[Bonjour](../gateway/bonjour.md)のCoreDNS例を参照してください。

### 手動ホスト/ポート

設定で「**手動ホスト**」を有効にし、ゲートウェイのホストとポート（デフォルト`18789`）を入力します。

## Canvas + A2UI

iOSノードはWKWebViewキャンバスをレンダリングします。`node.invoke`を使用して操作します:

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.navigate --params '{"url":"http://<gateway-host>:18789/__openclaw__/canvas/"}'
```

注意点:

-   Gatewayキャンバスホストは`/__openclaw__/canvas/`と`/__openclaw__/a2ui/`を提供します。
-   これはGateway HTTPサーバー（`gateway.port`と同じポート、デフォルト`18789`）から提供されます。
-   iOSノードは、キャンバスホストURLがアドバタイズされている場合、接続時に自動的にA2UIにナビゲートします。
-   組み込みのスキャフォールドに戻るには、`canvas.navigate`と`{"url":""}`を使用します。

### Canvas eval / スナップショット

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.eval --params '{"javaScript":"(() => { const {ctx} = window.__openclaw; ctx.clearRect(0,0,innerWidth,innerHeight); ctx.lineWidth=6; ctx.strokeStyle=\"#ff2d55\"; ctx.beginPath(); ctx.moveTo(40,40); ctx.lineTo(innerWidth-40, innerHeight-40); ctx.stroke(); return \"ok\"; })()"}'
```

```bash
openclaw nodes invoke --node "iOS Node" --command canvas.snapshot --params '{"maxWidth":900,"format":"jpeg"}'
```

## ボイスウェイク + トークモード

-   ボイスウェイクとトークモードは設定で利用可能です。
-   iOSはバックグラウンドオーディオをサスペンドする場合があります。アプリがアクティブでないときの音声機能はベストエフォートとして扱ってください。

## 一般的なエラー

-   `NODE_BACKGROUND_UNAVAILABLE`: iOSアプリをフォアグラウンドにしてください（キャンバス/カメラ/画面コマンドにはこれが必要です）。
-   `A2UI_HOST_NOT_CONFIGURED`: GatewayがキャンバスホストURLをアドバタイズしていません。[Gateway設定](../gateway/configuration.md)の`canvasHost`を確認してください。
-   ペアリングプロンプトが表示されない: `openclaw devices list`を実行し、手動で承認してください。
-   再インストール後、再接続に失敗する: Keychainのペアリングトークンがクリアされました。ノードを再ペアリングしてください。

## 関連ドキュメント

-   [ペアリング](../channels/pairing.md)
-   [ディスカバリー](../gateway/discovery.md)
-   [Bonjour](../gateway/bonjour.md)

[Androidアプリ](./android.md)[DigitalOcean](./digitalocean.md)