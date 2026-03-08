

  ネットワーキングとディスカバリー

  
# ネットワークモデル

ほとんどの操作は、Gateway (`openclaw gateway`) を経由して流れます。これは、チャネル接続とWebSocketコントロールプレーンを所有する単一の長期間実行プロセスです。

## コアルール

-   ホストごとに1つのGatewayを推奨します。これはWhatsApp Webセッションを所有することが許可されている唯一のプロセスです。レスキューボットや厳密な分離が必要な場合は、分離されたプロファイルとポートで複数のゲートウェイを実行してください。詳細は [複数ゲートウェイ](./multiple-gateways.md) を参照してください。
-   ループバック優先: Gateway WS はデフォルトで `ws://127.0.0.1:18789` を使用します。ウィザードは、ループバックの場合でもデフォルトでゲートウェイトークンを生成します。Tailnetアクセスの場合は、`openclaw gateway --bind tailnet --token ...` を実行してください。なぜなら、ループバック以外のバインドにはトークンが必要だからです。
-   ノードは、必要に応じてLAN、tailnet、またはSSHを介してGateway WSに接続します。従来のTCPブリッジは非推奨です。
-   キャンバスホストは、Gateway HTTPサーバーによって、Gatewayと同じポート（デフォルト `18789`）で提供されます:
    -   `/__openclaw__/canvas/`
    -   `/__openclaw__/a2ui/` `gateway.auth` が設定され、Gatewayがループバックを超えてバインドされている場合、これらのルートはGateway認証によって保護されます。ノードクライアントは、アクティブなWSセッションに紐づくノードスコープのキャパビリティURLを使用します。詳細は [Gateway設定](./configuration.md) (`canvasHost`, `gateway`) を参照してください。
-   リモート使用は、通常SSHトンネルまたはtailnet VPNです。詳細は [リモートアクセス](./remote.md) と [ディスカバリー](./discovery.md) を参照してください。

[ローカルモデル](./local-models.md)[Gateway所有ペアリング](./pairing.md)