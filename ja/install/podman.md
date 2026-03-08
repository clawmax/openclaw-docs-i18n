

  その他のインストール方法

  
# Podman

OpenClawゲートウェイを**rootless** Podmanコンテナで実行します。Dockerと同じイメージを使用します（リポジトリの[Dockerfile](https://github.com/openclaw/openclaw/blob/main/Dockerfile)からビルド）。

## 必要条件

-   Podman (rootless)
-   ワンタイムセットアップ用のSudo（ユーザー作成、イメージビルド）

## クイックスタート

**1\. ワンタイムセットアップ**（リポジトリルートから実行；ユーザー作成、イメージビルド、起動スクリプトインストール）：

```
./setup-podman.sh
```

これにより、ゲートウェイがウィザードを実行せずに起動できるように、最小限の`~openclaw/.openclaw/openclaw.json`（`gateway.mode="local"`を設定）も作成されます。デフォルトでは、コンテナは**systemdサービスとしてインストールされません**。手動で起動します（下記参照）。自動起動と再起動を備えた本番環境スタイルのセットアップには、代わりにsystemd Quadletユーザーサービスとしてインストールしてください：

```bash
./setup-podman.sh --quadlet
```

（または`OPENCLAW_PODMAN_QUADLET=1`を設定；`--container`を使用すると、コンテナと起動スクリプトのみをインストールします。）オプションのビルド時環境変数（`setup-podman.sh`実行前に設定）：

-   `OPENCLAW_DOCKER_APT_PACKAGES` — イメージビルド中に追加のaptパッケージをインストール
-   `OPENCLAW_EXTENSIONS` — 拡張機能の依存関係を事前インストール（スペース区切りの拡張機能名、例：`diagnostics-otel matrix`）

**2\. ゲートウェイ起動**（手動、簡単な動作確認用）：

```bash
./scripts/run-openclaw-podman.sh launch
```

**3\. オンボーディングウィザード**（例：チャネルやプロバイダーを追加する場合）：

```bash
./scripts/run-openclaw-podman.sh launch setup
```

その後、`http://127.0.0.1:18789/`を開き、`~openclaw/.openclaw/.env`からのトークン（またはセットアップで表示された値）を使用します。

## Systemd (Quadlet, オプション)

`./setup-podman.sh --quadlet`（または`OPENCLAW_PODMAN_QUADLET=1`）を実行した場合、[Podman Quadlet](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)ユニットがインストールされ、ゲートウェイはopenclawユーザーのsystemdユーザーサービスとして実行されます。サービスは有効化され、セットアップ終了時に起動します。

-   **起動:** `sudo systemctl --machine openclaw@ --user start openclaw.service`
-   **停止:** `sudo systemctl --machine openclaw@ --user stop openclaw.service`
-   **状態:** `sudo systemctl --machine openclaw@ --user status openclaw.service`
-   **ログ:** `sudo journalctl --machine openclaw@ --user -u openclaw.service -f`

quadletファイルは`~openclaw/.config/containers/systemd/openclaw.container`に配置されます。ポートや環境変数を変更するには、そのファイル（またはそれが参照する`.env`）を編集し、`sudo systemctl --machine openclaw@ --user daemon-reload`を実行してサービスを再起動します。起動時には、openclawに対してlingeringが有効になっている場合（loginctlが利用可能な場合、セットアップでこれを行います）、サービスは自動的に起動します。Quadletを**後から**追加するには（最初のセットアップで使用しなかった場合）、再実行します：`./setup-podman.sh --quadlet`。

## openclawユーザー (非ログイン)

`setup-podman.sh`は専用のシステムユーザー`openclaw`を作成します：

-   **シェル:** `nologin` — 対話型ログイン不可；攻撃対象領域を減らします。
-   **ホーム:** 例：`/home/openclaw` — `~/.openclaw`（設定、ワークスペース）と起動スクリプト`run-openclaw-podman.sh`を保持します。
-   **Rootless Podman:** ユーザーは**subuid**と**subgid**の範囲を持っている必要があります。多くのディストリビューションでは、ユーザー作成時にこれらが自動的に割り当てられます。セットアップで警告が表示された場合は、`/etc/subuid`と`/etc/subgid`に行を追加します：
    
    コピー
    
    ```
    openclaw:100000:65536
    ```
    
    その後、そのユーザーとしてゲートウェイを起動します（例：cronやsystemdから）：
    
    コピー
    
    ```bash
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh
    sudo -u openclaw /home/openclaw/run-openclaw-podman.sh setup
    ```
    
-   **設定:** `openclaw`とrootのみが`/home/openclaw/.openclaw`にアクセスできます。設定を編集するには：ゲートウェイ実行後にControl UIを使用するか、`sudo -u openclaw $EDITOR /home/openclaw/.openclaw/openclaw.json`を実行します。

## 環境と設定

-   **トークン:** `~openclaw/.openclaw/.env`に`OPENCLAW_GATEWAY_TOKEN`として保存されます。`setup-podman.sh`と`run-openclaw-podman.sh`は、欠落している場合に生成します（`openssl`、`python3`、または`od`を使用）。
-   **オプション:** その`.env`でプロバイダーキー（例：`GROQ_API_KEY`、`OLLAMA_API_KEY`）やその他のOpenClaw環境変数を設定できます。
-   **ホストポート:** デフォルトでは、スクリプトは`18789`（ゲートウェイ）と`18790`（ブリッジ）をマッピングします。起動時に`OPENCLAW_PODMAN_GATEWAY_HOST_PORT`と`OPENCLAW_PODMAN_BRIDGE_HOST_PORT`を使用して**ホスト**側のポートマッピングを上書きできます。
-   **ゲートウェイバインド:** デフォルトでは、`run-openclaw-podman.sh`は安全なローカルアクセスのために`--bind loopback`でゲートウェイを起動します。LANに公開するには、`OPENCLAW_GATEWAY_BIND=lan`を設定し、`openclaw.json`で`gateway.controlUi.allowedOrigins`を設定するか（または明示的にホストヘッダーフォールバックを有効にします）。
-   **パス:** ホストの設定とワークスペースは、デフォルトで`~openclaw/.openclaw`と`~openclaw/.openclaw/workspace`です。起動スクリプトが使用するホストパスは、`OPENCLAW_CONFIG_DIR`と`OPENCLAW_WORKSPACE_DIR`で上書きできます。

## ストレージモデル

-   **永続的なホストデータ:** `OPENCLAW_CONFIG_DIR`と`OPENCLAW_WORKSPACE_DIR`はコンテナにバインドマウントされ、ホスト上で状態を保持します。
-   **一時的なサンドボックスtmpfs:** `agents.defaults.sandbox`を有効にした場合、ツールサンドボックスコンテナは`/tmp`、`/var/tmp`、`/run`に`tmpfs`をマウントします。これらのパスはメモリバックアップされ、サンドボックスコンテナと共に消滅します。トップレベルのPodmanコンテナセットアップは独自のtmpfsマウントを追加しません。
-   **ディスク使用量増加のホットスポット:** 監視すべき主なパスは、`media/`、`agents//sessions/sessions.json`、トランスクリプトJSONLファイル、`cron/runs/*.jsonl`、および`/tmp/openclaw/`（または設定した`logging.file`）下のローテーションするファイルログです。

`setup-podman.sh`は現在、イメージtarをプライベート一時ディレクトリにステージングし、セットアップ中に選択されたベースディレクトリを表示します。非root実行では、そのベースが安全に使用できる場合にのみ`TMPDIR`を受け入れます。それ以外の場合は、`/var/tmp`、次に`/tmp`にフォールバックします。保存されたtarは所有者のみがアクセス可能なまま維持され、ターゲットユーザーの`podman load`にストリーミングされるため、プライベートな呼び出し元の一時ディレクトリがセットアップを妨げることはありません。

## 便利なコマンド

-   **ログ:** Quadlet使用時：`sudo journalctl --machine openclaw@ --user -u openclaw.service -f`。スクリプト使用時：`sudo -u openclaw podman logs -f openclaw`
-   **停止:** Quadlet使用時：`sudo systemctl --machine openclaw@ --user stop openclaw.service`。スクリプト使用時：`sudo -u openclaw podman stop openclaw`
-   **再起動:** Quadlet使用時：`sudo systemctl --machine openclaw@ --user start openclaw.service`。スクリプト使用時：起動スクリプトを再実行するか、`podman start openclaw`
-   **コンテナ削除:** `sudo -u openclaw podman rm -f openclaw` — ホスト上の設定とワークスペースは保持されます

## トラブルシューティング

-   **設定またはauth-profilesで権限拒否 (EACCES):** コンテナはデフォルトで`--userns=keep-id`を使用し、スクリプトを実行するホストユーザーと同じuid/gidで実行されます。ホストの`OPENCLAW_CONFIG_DIR`と`OPENCLAW_WORKSPACE_DIR`がそのユーザーによって所有されていることを確認してください。
-   **ゲートウェイ起動がブロックされる (`gateway.mode=local`が欠落):** `~openclaw/.openclaw/openclaw.json`が存在し、`gateway.mode="local"`を設定していることを確認してください。`setup-podman.sh`はこのファイルが欠落している場合に作成します。
-   **ユーザーopenclawでRootless Podmanが失敗:** `/etc/subuid`と`/etc/subgid`に`openclaw`の行（例：`openclaw:100000:65536`）が含まれているか確認してください。欠落している場合は追加して再起動します。
-   **コンテナ名が使用中:** 起動スクリプトは`podman run --replace`を使用するため、既存のコンテナは再起動時に置き換えられます。手動でクリーンアップするには：`podman rm -f openclaw`。
-   **openclawとして実行時にスクリプトが見つからない:** `setup-podman.sh`が実行され、`run-openclaw-podman.sh`がopenclawのホーム（例：`/home/openclaw/run-openclaw-podman.sh`）にコピーされていることを確認してください。
-   **Quadletサービスが見つからない、または起動に失敗:** `.container`ファイルを編集した後、`sudo systemctl --machine openclaw@ --user daemon-reload`を実行してください。Quadletにはcgroups v2が必要です：`podman info --format '{{.Host.CgroupsVersion}}'`は`2`を表示するはずです。

## オプション: 自分のユーザーとして実行

通常のユーザーとしてゲートウェイを実行するには（専用のopenclawユーザーなし）：イメージをビルドし、`OPENCLAW_GATEWAY_TOKEN`を含む`~/.openclaw/.env`を作成し、`--userns=keep-id`とホームの`~/.openclaw`へのマウントを指定してコンテナを実行します。起動スクリプトはopenclawユーザーフロー用に設計されています。シングルユーザーセットアップでは、代わりにスクリプトからの`podman run`コマンドを手動で実行し、設定とワークスペースをホームディレクトリに向けることができます。ほとんどのユーザーには、`setup-podman.sh`を使用し、openclawユーザーとして実行して設定とプロセスを分離することをお勧めします。

[Docker](./docker.md)[Nix](./nix.md)