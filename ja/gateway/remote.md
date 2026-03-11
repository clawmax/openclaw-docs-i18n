

  リモートアクセス

  
# リモートアクセス

このリポジトリは、専用ホスト（デスクトップ/サーバー）で単一のGateway（マスター）を実行し続け、クライアントをそれに接続することで、「SSH経由のリモート」をサポートします。

-   **オペレーター（あなた / macOSアプリ）向け**: SSHトンネリングはユニバーサルなフォールバックです。
-   **ノード（iOS/Androidおよび将来のデバイス）向け**: Gateway **WebSocket** に接続します（LAN/テールネットまたは必要に応じてSSHトンネル経由）。

## 核となる考え方

-   Gateway WebSocketは、設定されたポート（デフォルトは18789）で**ループバック**にバインドされます。
-   リモート使用のためには、そのループバックポートをSSH経由で転送します（またはテールネット/VPNを使用し、トンネリングを減らします）。

## 一般的なVPN/テールネット設定（エージェントが存在する場所）

**Gatewayホスト**を「エージェントが存在する場所」と考えてください。それはセッション、認証プロファイル、チャネル、状態を所有します。あなたのラップトップ/デスクトップ（およびノード）はそのホストに接続します。

### 1) テールネット内の常時稼働Gateway（VPSまたはホームサーバー）

永続的なホストでGatewayを実行し、**Tailscale**またはSSH経由で到達します。

-   **最適なUX:** `gateway.bind: "loopback"` を維持し、Control UIには **Tailscale Serve** を使用します。
-   **フォールバック:** ループバックを維持し、アクセスが必要なマシンからSSHトンネルを使用します。
-   **例:** [exe.dev](../install/exe-dev.md)（簡単なVM）または [Hetzner](../install/hetzner.md)（本番用VPS）。

これは、ラップトップが頻繁にスリープするが、エージェントを常時稼働させたい場合に理想的です。

### 2) ホームデスクトップがGatewayを実行、ラップトップはリモートコントロール

ラップトップはエージェントを実行**しません**。リモートで接続します:

-   macOSアプリの **Remote over SSH** モードを使用します（設定 → 一般 → "OpenClaw runs"）。
-   アプリがトンネルを開き管理するため、WebChat + ヘルスチェックが「そのまま動作します」。

手順書: [macOSリモートアクセス](../platforms/mac/remote.md)。

### 3) ラップトップがGatewayを実行、他のマシンからリモートアクセス

Gatewayをローカルに維持し、安全に公開します:

-   他のマシンからラップトップへのSSHトンネル、または
-   Control UIをTailscale Serveし、Gatewayをループバック専用に維持します。

ガイド: [Tailscale](./tailscale.md) および [Web概要](../web.md)。

## コマンドフロー（何がどこで実行されるか）

1つのゲートウェイサービスが状態 + チャネルを所有します。ノードは周辺機器です。フローの例（Telegram → ノード）:

-   Telegramメッセージが **Gateway** に到着します。
-   Gatewayは **エージェント** を実行し、ノードツールを呼び出すかどうかを決定します。
-   GatewayはGateway WebSocket経由で **ノード** を呼び出します（`node.*` RPC）。
-   ノードが結果を返します。GatewayはTelegramに返信します。

注意:

-   **ノードはゲートウェイサービスを実行しません。** ホストごとに1つのゲートウェイのみを実行するべきです（意図的に分離されたプロファイルを実行する場合を除く。 [複数のゲートウェイ](./multiple-gateways.md) を参照）。
-   macOSアプリの「ノードモード」は、Gateway WebSocketを介した単なるノードクライアントです。

## SSHトンネル（CLI + ツール）

リモートGateway WSへのローカルトンネルを作成します:

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

トンネルが確立されると:

-   `openclaw health` および `openclaw status --deep` は、`ws://127.0.0.1:18789` を介してリモートゲートウェイに到達します。
-   `openclaw gateway {status,health,send,agent,call}` も、必要に応じて `--url` を介して転送されたURLをターゲットにできます。

注: `18789` を設定した `gateway.port`（または `--port`/`OPENCLAW_GATEWAY_PORT`）に置き換えてください。注: `--url` を渡す場合、CLIは設定または環境認証情報にフォールバックしません。`--token` または `--password` を明示的に含めてください。明示的な認証情報がない場合はエラーになります。

## CLIリモートデフォルト

リモートターゲットを永続化して、CLIコマンドがデフォルトでそれを使用するようにできます:

```json
{
  gateway: {
    mode: "remote",
    remote: {
      url: "ws://127.0.0.1:18789",
      token: "your-token",
    },
  },
}
```

ゲートウェイがループバック専用の場合、URLを `ws://127.0.0.1:18789` に維持し、最初にSSHトンネルを開きます。

## 認証情報の優先順位

Gateway呼び出し/プローブの認証情報解決は、現在、1つの共有契約に従います:

-   明示的な認証情報（`--token`、`--password`、またはツールの `gatewayToken`）が常に優先されます。
-   ローカルモードのデフォルト:
    -   token: `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token` -> `gateway.remote.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.auth.password` -> `gateway.remote.password`
-   リモートモードのデフォルト:
    -   token: `gateway.remote.token` -> `OPENCLAW_GATEWAY_TOKEN` -> `gateway.auth.token`
    -   password: `OPENCLAW_GATEWAY_PASSWORD` -> `gateway.remote.password` -> `gateway.auth.password`
-   リモートプローブ/ステータスのトークンチェックはデフォルトで厳密です: リモートモードをターゲットにする場合、`gateway.remote.token` のみを使用します（ローカルトークンのフォールバックなし）。
-   レガシーな `CLAWDBOT_GATEWAY_*` 環境変数は、互換性呼び出しパスでのみ使用されます。プローブ/ステータス/認証解決では `OPENCLAW_GATEWAY_*` のみを使用します。

## SSH経由のチャットUI

WebChatはもはや別のHTTPポートを使用しません。SwiftUIチャットUIはGateway WebSocketに直接接続します。

-   SSH経由で `18789` を転送します（上記参照）。その後、クライアントを `ws://127.0.0.1:18789` に接続します。
-   macOSでは、アプリの「Remote over SSH」モードを優先します。これはトンネルを自動的に管理します。

## macOSアプリ「Remote over SSH」

macOSメニューバーアプリは、同じセットアップをエンドツーエンドで駆動できます（リモートステータスチェック、WebChat、およびVoice Wake転送）。手順書: [macOSリモートアクセス](../platforms/mac/remote.md)。

## セキュリティルール（リモート/VPN）

短いバージョン: **バインドが必要だと確信していない限り、Gatewayをループバック専用に維持してください。**

-   **ループバック + SSH/Tailscale Serve** が最も安全なデフォルトです（公開されません）。
-   プレーンテキスト `ws://` はデフォルトでループバック専用です。信頼されたプライベートネットワークでは、ブレイクグラスとしてクライアントプロセスで `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` を設定します。
-   **非ループバックバインド**（`lan`/`tailnet`/`custom`、またはループバックが利用できない場合の `auto`）は、認証トークン/パスワードを使用する必要があります。
-   `gateway.remote.token` / `.password` はクライアント認証情報ソースです。これらだけではサーバー認証を構成**しません**。
-   ローカル呼び出しパスは、`gateway.auth.*` が設定されていない場合、`gateway.remote.*` をフォールバックとして使用できます。
-   `gateway.remote.tlsFingerprint` は、`wss://` を使用する場合にリモートTLS証明書をピン留めします。
-   **Tailscale Serve** は、`gateway.auth.allowTailscale: true` の場合、IDヘッダーを介してControl UI/WebSocketトラフィックを認証できます。HTTP APIエンドポイントは依然としてトークン/パスワード認証を必要とします。このトークンレスフローは、ゲートウェイホストが信頼されていることを前提としています。どこでもトークン/パスワードが必要な場合は、これを `false` に設定してください。
-   ブラウザコントロールはオペレーターアクセスと同様に扱います: テールネット専用 + 意図的なノードペアリング。

詳細: [セキュリティ](./security.md)。

[Bonjour Discovery](./bonjour.md)[リモートゲートウェイ設定](./remote-gateway-readme.md)