

  ネットワーキングとディスカバリ

  
# Bonjour Discovery

OpenClawは、アクティブなゲートウェイ（WebSocketエンドポイント）を発見するための**LAN限定の利便性**としてBonjour (mDNS / DNS‑SD)を使用します。これはベストエフォートであり、SSHやTailnetベースの接続性を**置き換えるものではありません**。

## Tailscale経由の広域Bonjour (ユニキャストDNS‑SD)

ノードとゲートウェイが異なるネットワーク上にある場合、マルチキャストmDNSは境界を越えません。Tailscale経由の**ユニキャストDNS‑SD**（「広域Bonjour」）に切り替えることで、同じディスカバリUXを維持できます。概要手順:

1.  ゲートウェイホスト（Tailnet経由で到達可能）でDNSサーバーを実行します。
2.  専用ゾーン（例: `openclaw.internal.`）の下に`_openclaw-gw._tcp`のDNS‑SDレコードを公開します。
3.  Tailscaleの**スプリットDNS**を設定し、選択したドメインがクライアント（iOSを含む）に対してそのDNSサーバー経由で解決されるようにします。

OpenClawは任意のディスカバリドメインをサポートします。`openclaw.internal.`は単なる例です。iOS/Androidノードは`local.`と設定した広域ドメインの両方を参照します。

### ゲートウェイ設定（推奨）

```json
{
  gateway: { bind: "tailnet" }, // tailnet専用 (推奨)
  discovery: { wideArea: { enabled: true } }, // 広域DNS-SD公開を有効化
}
```

### 一度限りのDNSサーバーセットアップ（ゲートウェイホスト）

```bash
openclaw dns setup --apply
```

これはCoreDNSをインストールし、以下のように設定します:

-   ポート53をゲートウェイのTailscaleインターフェースのみでリッスン
-   選択したドメイン（例: `openclaw.internal.`）を`~/.openclaw/dns/.db`から提供

Tailnet接続されたマシンから検証:

```bash
dns-sd -B _openclaw-gw._tcp openclaw.internal.
dig @<TAILNET_IPV4> -p 53 _openclaw-gw._tcp.openclaw.internal PTR +short
```

### Tailscale DNS設定

Tailscale管理コンソールで:

-   ゲートウェイのtailnet IP（UDP/TCP 53）を指すネームサーバーを追加します。
-   スプリットDNSを追加し、ディスカバリドメインがそのネームサーバーを使用するようにします。

クライアントがtailnet DNSを受け入れると、iOSノードはマルチキャストなしでディスカバリドメイン内の`_openclaw-gw._tcp`を参照できるようになります。

### ゲートウェイリスナーセキュリティ（推奨）

ゲートウェイWSポート（デフォルト`18789`）はデフォルトでループバックにバインドされます。LAN/tailnetアクセスのためには、明示的にバインドし、認証を有効にしたままにします。tailnet専用セットアップの場合:

-   `~/.openclaw/openclaw.json`で`gateway.bind: "tailnet"`を設定します。
-   ゲートウェイを再起動します（またはmacOSメニューバーアプリを再起動します）。

## 何がアドバタイズされるか

ゲートウェイのみが`_openclaw-gw._tcp`をアドバタイズします。

## サービスタイプ

-   `_openclaw-gw._tcp` — ゲートウェイトランスポートビーコン（macOS/iOS/Androidノードで使用）。

## TXTキー（非秘密のヒント）

ゲートウェイは、UIフローを便利にするための小さな非秘密のヒントをアドバタイズします:

-   `role=gateway`
-   `displayName=<フレンドリー名>`
-   `lanHost=<ホスト名>.local`
-   `gatewayPort=<ポート>` (ゲートウェイWS + HTTP)
-   `gatewayTls=1` (TLSが有効な場合のみ)
-   `gatewayTlsSha256=` (TLSが有効でフィンガープリントが利用可能な場合のみ)
-   `canvasPort=<ポート>` (キャンバスホストが有効な場合のみ; 現在は`gatewayPort`と同じ)
-   `sshPort=<ポート>` (上書きされない場合、デフォルトは22)
-   `transport=gateway`
-   `cliPath=` (オプション; 実行可能な`openclaw`エントリーポイントへの絶対パス)
-   `tailnetDns=` (オプションのヒント、Tailnetが利用可能な場合)

セキュリティ上の注意:

-   Bonjour/mDNS TXTレコードは**認証されていません**。クライアントはTXTを権威あるルーティング情報として扱ってはいけません。
-   クライアントは解決されたサービスエンドポイント（SRV + A/AAAA）を使用してルーティングすべきです。`lanHost`、`tailnetDns`、`gatewayPort`、`gatewayTlsSha256`はヒントとしてのみ扱ってください。
-   TLSピニングは、アドバタイズされた`gatewayTlsSha256`が以前に保存されたピンを上書きすることを決して許可してはいけません。
-   iOS/Androidノードは、ディスカバリベースの直接接続を**TLSのみ**として扱い、初回のフィンガープリントを信頼する前に明示的なユーザー確認を要求すべきです。

## macOSでのデバッグ

便利な組み込みツール:

-   インスタンスを参照:
    
    コピー
    
    ```bash
    dns-sd -B _openclaw-gw._tcp local.
    ```
    
-   1つのインスタンスを解決（``を置き換え）:
    
    コピー
    
    ```bash
    dns-sd -L "<instance>" _openclaw-gw._tcp local.
    ```
    

参照が機能しても解決が失敗する場合、通常はLANポリシーまたはmDNSリゾルバの問題に遭遇しています。

## ゲートウェイログでのデバッグ

ゲートウェイはローテーションするログファイルを書き込みます（起動時に`gateway log file: ...`として出力されます）。`bonjour:`行、特に以下を探してください:

-   `bonjour: advertise failed ...`
-   `bonjour: ... name conflict resolved` / `hostname conflict resolved`
-   `bonjour: watchdog detected non-announced service ...`

## iOSノードでのデバッグ

iOSノードは`NWBrowser`を使用して`_openclaw-gw._tcp`を発見します。ログをキャプチャするには:

-   設定 → ゲートウェイ → 詳細 → **Discovery Debug Logs**
-   設定 → ゲートウェイ → 詳細 → **Discovery Logs** → 再現 → **コピー**

ログにはブラウザの状態遷移と結果セットの変更が含まれます。

## 一般的な障害モード

-   **Bonjourはネットワークを越えない**: TailnetまたはSSHを使用してください。
-   **マルチキャストがブロックされている**: 一部のWi‑FiネットワークはmDNSを無効にしています。
-   **スリープ / インターフェースの変動**: macOSは一時的にmDNS結果をドロップすることがあります。再試行してください。
-   **参照は機能するが解決が失敗する**: マシン名をシンプルに保ち（絵文字や句読点を避け）、ゲートウェイを再起動してください。サービスインスタンス名はホスト名から派生するため、複雑すぎる名前は一部のリゾルバを混乱させることがあります。

## エスケープされたインスタンス名 (\\032)

Bonjour/DNS‑SDは、サービスインスタンス名内のバイトを10進数の`\DDD`シーケンスとしてエスケープすることがよくあります（例: スペースは`\032`になります）。

-   これはプロトコルレベルでは正常です。
-   UIは表示用にデコードすべきです（iOSは`BonjourEscapes.decode`を使用します）。

## 無効化 / 設定

-   `OPENCLAW_DISABLE_BONJOUR=1`はアドバタイズを無効にします（レガシー: `OPENCLAW_DISABLE_BONJOUR`）。
-   `~/.openclaw/openclaw.json`内の`gateway.bind`はゲートウェイのバインドモードを制御します。
-   `OPENCLAW_SSH_PORT`はTXTでアドバタイズされるSSHポートを上書きします（レガシー: `OPENCLAW_SSH_PORT`）。
-   `OPENCLAW_TAILNET_DNS`はTXTにMagicDNSヒントを公開します（レガシー: `OPENCLAW_TAILNET_DNS`）。
-   `OPENCLAW_CLI_PATH`はアドバタイズされるCLIパスを上書きします（レガシー: `OPENCLAW_CLI_PATH`）。

## 関連ドキュメント

-   ディスカバリポリシーとトランスポート選択: [Discovery](./discovery.md)
-   ノードペアリング + 承認: [Gateway pairing](./pairing.md)

[Discovery and Transports](./discovery.md)[Remote Access](./remote.md)