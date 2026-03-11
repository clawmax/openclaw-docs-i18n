

  ホスティングとデプロイ

  
# exe.dev

目標: exe.dev VM上で動作するOpenClaw Gatewayを、あなたのラップトップから `https://<vm-name>.exe.xyz` でアクセス可能にする。このページはexe.devのデフォルトの **exeuntu** イメージを前提としています。別のディストリビューションを選択した場合は、パッケージを適宜読み替えてください。

## 初心者向けクイックパス

1.  [https://exe.new/openclaw](https://exe.new/openclaw)
2.  必要に応じて認証キー/トークンを入力
3.  VMの横にある「Agent」をクリックし、待機…
4.  ???
5.  完了

## 必要なもの

-   exe.devアカウント
-   [exe.dev](https://exe.dev)仮想マシンへの `ssh exe.dev` アクセス（オプション）

## Shelleyによる自動インストール

[exe.dev](https://exe.dev)のエージェントであるShelleyは、私たちのプロンプトを使ってOpenClawを即座にインストールできます。使用されるプロンプトは以下の通りです：

```bash
Set up OpenClaw (https://docs.openclaw.ai/install) on this VM. Use the non-interactive and accept-risk flags for openclaw onboarding. Add the supplied auth or token as needed. Configure nginx to forward from the default port 18789 to the root location on the default enabled site config, making sure to enable Websocket support. Pairing is done by "openclaw devices list" and "openclaw devices approve <request id>". Make sure the dashboard shows that OpenClaw's health is OK. exe.dev handles forwarding from port 8000 to port 80/443 and HTTPS for us, so the final "reachable" should be <vm-name>.exe.xyz, without port specification.
```

## 手動インストール

## 1) VMを作成する

あなたのデバイスから：

```bash
ssh exe.dev new
```

次に接続：

```bash
ssh <vm-name>.exe.xyz
```

ヒント：このVMを **ステートフル** のままにしてください。OpenClawは状態を `~/.openclaw/` と `~/.openclaw/workspace/` の下に保存します。

## 2) 前提条件をインストールする（VM上）

```bash
sudo apt-get update
sudo apt-get install -y git curl jq ca-certificates openssl
```

## 3) OpenClawをインストールする

OpenClawインストールスクリプトを実行：

```bash
curl -fsSL https://openclaw.ai/install.sh | bash
```

## 4) nginxを設定してOpenClawをポート8000にプロキシする

`/etc/nginx/sites-enabled/default` を以下の内容で編集：

```nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    listen 8000;
    listen [::]:8000;

    server_name _;

    location / {
        proxy_pass http://127.0.0.1:18789;
        proxy_http_version 1.1;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Standard proxy headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Timeout settings for long-lived connections
        proxy_read_timeout 86400s;
        proxy_send_timeout 86400s;
    }
}
```

## 5) OpenClawにアクセスして権限を付与する

`https://<vm-name>.exe.xyz/` にアクセスします（オンボーディングからのControl UI出力を参照）。認証を求められた場合は、VM上の `gateway.auth.token` からトークンを貼り付けます（`openclaw config get gateway.auth.token` で取得、または `openclaw doctor --generate-gateway-token` で生成）。デバイスは `openclaw devices list` と `openclaw devices approve ` で承認します。わからない場合は、ブラウザからShelleyを使ってください！

## リモートアクセス

リモートアクセスは[exe.dev](https://exe.dev)の認証によって処理されます。デフォルトでは、ポート8000からのHTTPトラフィックはメール認証付きで `https://<vm-name>.exe.xyz` に転送されます。

## アップデート

```bash
npm i -g openclaw@latest
openclaw doctor
openclaw gateway restart
openclaw health
```

ガイド：[アップデート](./updating.md)

[macOS VM](./macos-vm.md)[Railwayにデプロイ](./railway.md)