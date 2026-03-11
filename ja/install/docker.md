

  その他のインストール方法

  
# Docker

Dockerは**オプション**です。コンテナ化ゲートウェイが必要な場合、またはDockerフローを検証したい場合にのみ使用してください。

## Dockerは私に適していますか？

-   **はい**: 分離された使い捨てのゲートウェイ環境が必要な場合、またはローカルインストールなしでホスト上でOpenClawを実行したい場合。
-   **いいえ**: 自身のマシンで実行しており、最も高速な開発ループを望む場合。代わりに通常のインストールフローを使用してください。
-   **サンドボックス化に関する注意**: エージェントサンドボックス化もDockerを使用しますが、**ゲートウェイ全体をDocker内で実行する必要はありません**。詳細は[サンドボックス化](../gateway/sandboxing.md)を参照してください。

このガイドでは以下を扱います:

-   コンテナ化ゲートウェイ (Docker内の完全なOpenClaw)
-   セッションごとのエージェントサンドボックス (ホストゲートウェイ + Dockerで分離されたエージェントツール)

サンドボックス化の詳細: [サンドボックス化](../gateway/sandboxing.md)

## 要件

-   Docker Desktop (またはDocker Engine) + Docker Compose v2
-   イメージビルド用に少なくとも2 GBのRAM (`pnpm install` が1 GBホストでOOMキルされ、終了コード137になる可能性があります)
-   イメージ + ログ用の十分なディスク容量
-   VPS/パブリックホストで実行する場合、[ネットワーク公開のためのセキュリティ強化](../gateway/security.md#04-network-exposure-bind--port--firewall)、特にDockerの `DOCKER-USER` ファイアウォールポリシーを確認してください。

## コンテナ化ゲートウェイ (Docker Compose)

### クイックスタート (推奨)

> **ℹ️** ここでのDockerのデフォルトは、ホストエイリアスではなくバインドモード (`lan`/`loopback`) を想定しています。`gateway.bind` ではホストエイリアス (`0.0.0.0` や `localhost` など) ではなく、バインドモードの値 (例: `lan` または `loopback`) を使用してください。

 リポジトリのルートから:

```
./docker-setup.sh
```

このスクリプトは以下を行います:

-   ゲートウェイイメージをローカルでビルド (または `OPENCLAW_IMAGE` が設定されている場合はリモートイメージをプル)
-   オンボーディングウィザードを実行
-   オプションのプロバイダーセットアップのヒントを表示
-   Docker Compose経由でゲートウェイを起動
-   ゲートウェイトークンを生成し `.env` に書き込み

オプションの環境変数:

-   `OPENCLAW_IMAGE` — ローカルビルドの代わりにリモートイメージを使用 (例: `ghcr.io/openclaw/openclaw:latest`)
-   `OPENCLAW_DOCKER_APT_PACKAGES` — ビルド中に追加のaptパッケージをインストール
-   `OPENCLAW_EXTENSIONS` — ビルド時に拡張機能の依存関係を事前インストール (スペース区切りの拡張機能名、例: `diagnostics-otel matrix`)
-   `OPENCLAW_EXTRA_MOUNTS` — 追加のホストバインドマウントを追加
-   `OPENCLAW_HOME_VOLUME` — `/home/node` を名前付きボリュームで永続化
-   `OPENCLAW_SANDBOX` — Dockerゲートウェイサンドボックスブートストラップをオプトイン。明示的な真値のみが有効化: `1`, `true`, `yes`, `on`
-   `OPENCLAW_INSTALL_DOCKER_CLI` — ローカルイメージビルドのためのビルド引数パススルー (`1` はイメージ内にDocker CLIをインストール)。`docker-setup.sh` はローカルビルドで `OPENCLAW_SANDBOX=1` の場合、これを自動的に設定します。
-   `OPENCLAW_DOCKER_SOCKET` — Dockerソケットパスを上書き (デフォルト: `DOCKER_HOST=unix://...` パス、それ以外の場合は `/var/run/docker.sock`)
-   `OPENCLAW_ALLOW_INSECURE_PRIVATE_WS=1` — 非常用: CLI/オンボーディングクライアントパス用の信頼されたプライベートネットワーク `ws://` ターゲットを許可 (デフォルトはループバックのみ)
-   `OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` — WebGL/3D互換性が必要な場合、コンテナブラウザの強化フラグ `--disable-3d-apis`, `--disable-software-rasterizer`, `--disable-gpu` を無効化。
-   `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` — ブラウザフローで拡張機能が必要な場合に拡張機能を有効に保つ (デフォルトではサンドボックスブラウザで拡張機能は無効化されます)。
-   `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` — Chromiumレンダラープロセス制限を設定; `0` に設定するとフラグをスキップしChromiumのデフォルト動作を使用。

完了後:

-   ブラウザで `http://127.0.0.1:18789/` を開きます。
-   トークンをControl UI (設定 → トークン) に貼り付けます。
-   URLを再度確認する必要がありますか? `docker compose run --rm openclaw-cli dashboard --no-open` を実行してください。

### Dockerゲートウェイのエージェントサンドボックスを有効化 (オプトイン)

`docker-setup.sh` はDockerデプロイメント用に `agents.defaults.sandbox.*` をブートストラップすることもできます。以下で有効化:

```bash
export OPENCLAW_SANDBOX=1
./docker-setup.sh
```

カスタムソケットパス (例: rootless Docker):

```bash
export OPENCLAW_SANDBOX=1
export OPENCLAW_DOCKER_SOCKET=/run/user/1000/docker.sock
./docker-setup.sh
```

注意:

-   スクリプトはサンドボックスの前提条件を通過した後にのみ `docker.sock` をマウントします。
-   サンドボックスセットアップが完了できない場合、スクリプトは再実行時の古い/壊れたサンドボックス設定を避けるため `agents.defaults.sandbox.mode` を `off` にリセットします。
-   `Dockerfile.sandbox` が欠落している場合、スクリプトは警告を表示して続行します; 必要に応じて `scripts/sandbox-setup.sh` で `openclaw-sandbox:bookworm-slim` をビルドしてください。
-   ローカル以外の `OPENCLAW_IMAGE` 値の場合、イメージにはサンドボックス実行用のDocker CLIサポートが既に含まれている必要があります。

### 自動化/CI (非対話型、TTYノイズなし)

スクリプトとCIでは、`-T` でCompose疑似TTY割り当てを無効化:

```bash
docker compose run -T --rm openclaw-cli gateway probe
docker compose run -T --rm openclaw-cli devices list --json
```

自動化でClaudeセッション変数をエクスポートしない場合、未設定のままにすると `docker-compose.yml` でデフォルトで空値に解決され、繰り返される「変数が設定されていません」警告を回避します。

### 共有ネットワークセキュリティに関する注意 (CLI + ゲートウェイ)

`openclaw-cli` は `network_mode: "service:openclaw-gateway"` を使用するため、CLIコマンドはDocker内で `127.0.0.1` 経由でゲートウェイに確実に到達できます。これを共有信頼境界として扱ってください: ループバックバインドはこれら2つのコンテナ間の分離ではありません。より強力な分離が必要な場合は、バンドルされた `openclaw-cli` サービスではなく、別のコンテナ/ホストネットワークパスからコマンドを実行してください。CLIプロセスが侵害された場合の影響を軽減するため、compose設定は `openclaw-cli` で `NET_RAW`/`NET_ADMIN` をドロップし、`no-new-privileges` を有効にします。ホスト上に設定/ワークスペースを書き込みます:

-   `~/.openclaw/`
-   `~/.openclaw/workspace`

VPSで実行していますか? [Hetzner (Docker VPS)](./hetzner.md) を参照してください。

### リモートイメージを使用 (ローカルビルドをスキップ)

公式の事前ビルドイメージは以下で公開されています:

-   [GitHub Container Registryパッケージ](https://github.com/openclaw/openclaw/pkgs/container/openclaw)

イメージ名 `ghcr.io/openclaw/openclaw` を使用してください (同様の名前のDocker Hubイメージではありません)。一般的なタグ:

-   `main` — `main` からの最新ビルド
-   `` — リリースタグビルド (例: `2026.2.26`)
-   `latest` — 最新の安定リリースタグ

### ベースイメージメタデータ

メインDockerイメージは現在以下を使用しています:

-   `node:22-bookworm`

dockerイメージは現在OCIベースイメージアノテーションを公開しています (sha256は例です):

-   `org.opencontainers.image.base.name=docker.io/library/node:22-bookworm`
-   `org.opencontainers.image.base.digest=sha256:6d735b4d33660225271fda0a412802746658c3a1b975507b2803ed299609760a`
-   `org.opencontainers.image.source=https://github.com/openclaw/openclaw`
-   `org.opencontainers.image.url=https://openclaw.ai`
-   `org.opencontainers.image.documentation=https://docs.openclaw.ai/install/docker`
-   `org.opencontainers.image.licenses=MIT`
-   `org.opencontainers.image.title=OpenClaw`
-   `org.opencontainers.image.description=OpenClawゲートウェイおよびCLIランタイムコンテナイメージ`
-   `org.opencontainers.image.revision=<git-sha>`
-   `org.opencontainers.image.version=<tag-or-main>`
-   `org.opencontainers.image.created=`

参照: [OCIイメージアノテーション](https://github.com/opencontainers/image-spec/blob/main/annotations.md) リリースコンテキスト: このリポジトリのタグ付き履歴は、`v2026.2.22` およびそれ以前の2026タグ (例: `v2026.2.21`, `v2026.2.9`) ですでにBookwormを使用しています。デフォルトでは、セットアップスクリプトはソースからイメージをビルドします。事前ビルドイメージをプルするには、スクリプト実行前に `OPENCLAW_IMAGE` を設定してください:

```bash
export OPENCLAW_IMAGE="ghcr.io/openclaw/openclaw:latest"
./docker-setup.sh
```

スクリプトは `OPENCLAW_IMAGE` がデフォルトの `openclaw:local` ではないことを検出し、`docker build` の代わりに `docker pull` を実行します。その他すべて (オンボーディング、ゲートウェイ起動、トークン生成) は同じように動作します。`docker-setup.sh` はローカルの `docker-compose.yml` とヘルパーファイルを使用するため、リポジトリルートから実行されます。`OPENCLAW_IMAGE` はローカルイメージビルド時間をスキップします; compose/セットアップワークフローを置き換えるものではありません。

### シェルヘルパー (オプション)

日々のDocker管理を容易にするため、`ClawDock` をインストール:

```bash
mkdir -p ~/.clawdock && curl -sL https://raw.githubusercontent.com/openclaw/openclaw/main/scripts/shell-helpers/clawdock-helpers.sh -o ~/.clawdock/clawdock-helpers.sh
```

**シェル設定に追加 (zsh):**

```bash
echo 'source ~/.clawdock/clawdock-helpers.sh' >> ~/.zshrc && source ~/.zshrc
```

その後、`clawdock-start`, `clawdock-stop`, `clawdock-dashboard` などを使用してください。すべてのコマンドは `clawdock-help` を実行してください。詳細は [`ClawDock` ヘルパーREADME](https://github.com/openclaw/openclaw/blob/main/scripts/shell-helpers/README.md) を参照してください。

### 手動フロー (compose)

```bash
docker build -t openclaw:local -f Dockerfile .
docker compose run --rm openclaw-cli onboard
docker compose up -d openclaw-gateway
```

注意: `docker compose ...` はリポジトリルートから実行してください。`OPENCLAW_EXTRA_MOUNTS` または `OPENCLAW_HOME_VOLUME` を有効にした場合、セットアップスクリプトは `docker-compose.extra.yml` を書き込みます; 他の場所でComposeを実行する際にこれを含めてください:

```bash
docker compose -f docker-compose.yml -f docker-compose.extra.yml <command>
```

### Control UIトークン + ペアリング (Docker)

「unauthorized」または「disconnected (1008): pairing required」が表示される場合、新しいダッシュボードリンクを取得し、ブラウザデバイスを承認してください:

```bash
docker compose run --rm openclaw-cli dashboard --no-open
docker compose run --rm openclaw-cli devices list
docker compose run --rm openclaw-cli devices approve <requestId>
```

詳細: [ダッシュボード](../web/dashboard.md), [デバイス](../cli/devices.md)。

### 追加マウント (オプション)

追加のホストディレクトリをコンテナにマウントしたい場合、`docker-setup.sh` 実行前に `OPENCLAW_EXTRA_MOUNTS` を設定してください。これはDockerバインドマウントのカンマ区切りリストを受け取り、`docker-compose.extra.yml` を生成することで `openclaw-gateway` と `openclaw-cli` の両方に適用します。例:

```bash
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意:

-   パスはmacOS/WindowsのDocker Desktopと共有されている必要があります。
-   各エントリはスペース、タブ、改行なしで `source:target[:options]` 形式である必要があります。
-   `OPENCLAW_EXTRA_MOUNTS` を編集した場合、追加のcomposeファイルを再生成するために `docker-setup.sh` を再実行してください。
-   `docker-compose.extra.yml` は生成されます。手動で編集しないでください。

### コンテナホーム全体を永続化 (オプション)

`/home/node` をコンテナ再作成後も永続化したい場合、`OPENCLAW_HOME_VOLUME` 経由で名前付きボリュームを設定してください。これによりDockerボリュームが作成され、標準のconfig/workspaceバインドマウントを維持しながら `/home/node` にマウントされます。ここでは名前付きボリュームを使用してください (バインドパスではありません); バインドマウントの場合は `OPENCLAW_EXTRA_MOUNTS` を使用してください。例:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

追加マウントと組み合わせることもできます:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
export OPENCLAW_EXTRA_MOUNTS="$HOME/.codex:/home/node/.codex:ro,$HOME/github:/home/node/github:rw"
./docker-setup.sh
```

注意:

-   名前付きボリュームは `^[A-Za-z0-9][A-Za-z0-9_.-]*$` に一致する必要があります。
-   `OPENCLAW_HOME_VOLUME` を変更した場合、追加のcomposeファイルを再生成するために `docker-setup.sh` を再実行してください。
-   名前付きボリュームは `docker volume rm ` で削除されるまで永続化されます。

### 追加のaptパッケージをインストール (オプション)

イメージ内にシステムパッケージ (ビルドツールやメディアライブラリなど) が必要な場合、`docker-setup.sh` 実行前に `OPENCLAW_DOCKER_APT_PACKAGES` を設定してください。これによりイメージビルド中にパッケージがインストールされ、コンテナが削除されても永続化されます。例:

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="ffmpeg build-essential"
./docker-setup.sh
```

注意:

-   これはaptパッケージ名のスペース区切りリストを受け入れます。
-   `OPENCLAW_DOCKER_APT_PACKAGES` を変更した場合、イメージを再ビルドするために `docker-setup.sh` を再実行してください。

### 拡張機能の依存関係を事前インストール (オプション)

独自の `package.json` を持つ拡張機能 (例: `diagnostics-otel`, `matrix`, `msteams`) は、初回ロード時にnpm依存関係をインストールします。代わりにそれらの依存関係をイメージに組み込みたい場合、`docker-setup.sh` 実行前に `OPENCLAW_EXTENSIONS` を設定してください:

```bash
export OPENCLAW_EXTENSIONS="diagnostics-otel matrix"
./docker-setup.sh
```

または直接ビルドする場合:

```bash
docker build --build-arg OPENCLAW_EXTENSIONS="diagnostics-otel matrix" .
```

注意:

-   これは拡張機能ディレクトリ名 (`extensions/` 下) のスペース区切りリストを受け入れます。
-   `package.json` を持つ拡張機能のみが影響を受けます; それを持たない軽量プラグインは無視されます。
-   `OPENCLAW_EXTENSIONS` を変更した場合、イメージを再ビルドするために `docker-setup.sh` を再実行してください。

### パワーユーザー / フル機能コンテナ (オプトイン)

デフォルトのDockerイメージは**セキュリティファースト**であり、非rootユーザー `node` として実行されます。これにより攻撃対象領域は小さくなりますが、以下を意味します:

-   実行時のシステムパッケージインストールなし
-   デフォルトでHomebrewなし
-   バンドルされたChromium/Playwrightブラウザなし

よりフル機能のコンテナが必要な場合、これらのオプトイン設定を使用してください:

1.  **`/home/node` を永続化**してブラウザダウンロードとツールキャッシュを保持:

```bash
export OPENCLAW_HOME_VOLUME="openclaw_home"
./docker-setup.sh
```

2.  **システム依存関係をイメージに組み込み** (再現可能 + 永続的):

```bash
export OPENCLAW_DOCKER_APT_PACKAGES="git curl jq"
./docker-setup.sh
```

3.  **`npx` なしでPlaywrightブラウザをインストール** (npmオーバーライドの競合を回避):

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

Playwrightがシステム依存関係をインストールする必要がある場合、実行時に `--with-deps` を使用する代わりに `OPENCLAW_DOCKER_APT_PACKAGES` でイメージを再ビルドしてください。

4.  **Playwrightブラウザダウンロードを永続化**:

-   `docker-compose.yml` で `PLAYWRIGHT_BROWSERS_PATH=/home/node/.cache/ms-playwright` を設定。
-   `OPENCLAW_HOME_VOLUME` 経由で `/home/node` が永続化されることを確認するか、`OPENCLAW_EXTRA_MOUNTS` 経由で `/home/node/.cache/ms-playwright` をマウント。

### 権限 + EACCES

イメージは `node` (uid 1000) として実行されます。`/home/node/.openclaw` で権限エラーが表示される場合、ホストバインドマウントがuid 1000によって所有されていることを確認してください。例 (Linuxホスト):

```bash
sudo chown -R 1000:1000 /path/to/openclaw-config /path/to/openclaw-workspace
```

利便性のためにrootとして実行することを選択した場合、セキュリティのトレードオフを受け入れます。

### より高速なリビルド (推奨)

リビルドを高速化するには、依存関係レイヤーがキャッシュされるようにDockerfileを順序付けます。これにより、ロックファイルが変更されない限り `pnpm install` の再実行を回避できます:

```dockerfile
FROM node:22-bookworm

# Bunをインストール (ビルドスクリプトに必須)
RUN curl -fsSL https://bun.sh/install | bash
ENV PATH="/root/.bun/bin:${PATH}"

RUN corepack enable

WORKDIR /app

# パッケージメタデータが変更されない限り依存関係をキャッシュ
COPY package.json pnpm-lock.yaml pnpm-workspace.yaml .npmrc ./
COPY ui/package.json ./ui/package.json
COPY scripts ./scripts

RUN pnpm install --frozen-lockfile

COPY . .
RUN pnpm build
RUN pnpm ui:install
RUN pnpm ui:build

ENV NODE_ENV=production

CMD ["node","dist/index.js"]
```

### チャネルセットアップ (オプション)

CLIコンテナを使用してチャネルを構成し、必要に応じてゲートウェイを再起動してください。WhatsApp (QR):

```bash
docker compose run --rm openclaw-cli channels login
```

Telegram (ボットトークン):

```bash
docker compose run --rm openclaw-cli channels add --channel telegram --token "<token>"
```

Discord (ボットトークン):

```bash
docker compose run --rm openclaw-cli channels add --channel discord --token "<token>"
```

ドキュメント: [WhatsApp](../channels/whatsapp.md), [Telegram](../channels/telegram.md), [Discord](../channels/discord.md)

### OpenAI Codex OAuth (ヘッドレスDocker)

ウィザードでOpenAI Codex OAuthを選択した場合、ブラウザURLが開き、`http://127.0.0.1:1455/auth/callback` でのコールバックをキャプチャしようとします。Dockerまたはヘッドレスセットアップでは、そのコールバックでブラウザエラーが表示される可能性があります。到達した完全なリダイレクトURLをコピーし、ウィザードに貼り付けて認証を完了してください。

### ヘルスチェック

コンテナプローブエンドポイント (認証不要):

```bash
curl -fsS http://127.0.0.1:18789/healthz
curl -fsS http://127.0.0.1:18789/readyz
```

エイリアス: `/health` および `/ready`。`/healthz` は「ゲートウェイプロセスが起動している」ための簡易的な生存性プローブです。`/readyz` は起動猶予期間中は準備完了状態を維持し、その後、必要な管理チャネルが猶予後も切断されているか、後で切断された場合にのみ `503` になります。Dockerイメージには `/healthz` をバックグラウンドでpingする組み込みの `HEALTHCHECK` が含まれています。平易に言えば: DockerはOpenClawがまだ応答するかどうかをチェックし続けます。チェックが失敗し続ける場合、Dockerはコンテナを `unhealthy` とマークし、オーケストレーションシステム (Docker Compose再起動ポリシー、Swarm、Kubernetesなど) が自動的に再起動または置換できます。認証済みの詳細ヘルススナップショット (ゲートウェイ + チャネル):

```bash
docker compose exec openclaw-gateway node dist/index.js health --token "$OPENCLAW_GATEWAY_TOKEN"
```

### E2Eスモークテスト (Docker)

```
scripts/e2e/onboard-docker.sh
```

### QRインポートスモークテスト (Docker)

```bash
pnpm test:docker:qr
```

### LAN対ループバック (Docker Compose)

`docker-setup.sh` はデフォルトで `OPENCLAW_GATEWAY_BIND=lan` とし、Dockerポート公開でホストアクセス `http://127.0.0.1:18789` が動作するようにします。

-   `lan` (デフォルト): ホストブラウザ + ホストCLIが公開されたゲートウェイポートに到達可能。
-   `loopback`: コンテナネットワーク名前空間内のプロセスのみがゲートウェイに直接到達可能; ホスト公開ポートアクセスは失敗する可能性があります。

セットアップスクリプトはまた、オンボーディング後に `gateway.mode=local` を固定し、Docker CLIコマンドがデフォルトでローカルループバックターゲットになるようにします。レガシー設定に関する注意: `gateway.bind` ではホストエイリアス (`0.0.0.0`, `127.0.0.1`, `localhost`, `::`, `::1`) ではなく、バインドモードの値 (`lan` / `loopback` / `custom` / `tailnet` / `auto`) を使用してください。Docker CLIコマンドから `Gateway target: ws://172.x.x.x:18789` または繰り返される `pairing required` エラーが表示される場合、以下を実行してください:

```bash
docker compose run --rm openclaw-cli config set gateway.mode local
docker compose run --rm openclaw-cli config set gateway.bind lan
docker compose run --rm openclaw-cli devices list --url ws://127.0.0.1:18789
```

### 注意事項

-   ゲートウェイバインドはコンテナ使用のためにデフォルトで `lan` (`OPENCLAW_GATEWAY_BIND`)。
-   Dockerfile CMDは `--allow-unconfigured` を使用; `gateway.mode` が `local` でないマウントされた設定でも起動します。ガードを強制するにはCMDを上書きしてください。
-   ゲートウェイコンテナはセッション (`~/.openclaw/agents//sessions/`) の信頼できる情報源です。

### ストレージモデル

-   **永続的なホストデータ:** Docker Composeは `OPENCLAW_CONFIG_DIR` を `/home/node/.openclaw` に、`OPENCLAW_WORKSPACE_DIR` を `/home/node/.openclaw/workspace` にバインドマウントするため、これらのパスはコンテナ置換後も存続します。
-   **一時的なサンドボックスtmpfs:** `agents.defaults.sandbox` が有効な場合、サンドボックスコンテナは `/tmp`、`/var/tmp`、`/run` に `tmpfs` を使用します。これらのマウントはトップレベルのComposeスタックとは別であり、サンドボックスコンテナと共に消滅します。
-   **ディスク増加のホットスポット:** `media/`、`agents//sessions/sessions.json`、トランスクリプトJSONLファイル、`cron/runs/*.jsonl`、および `/tmp/openclaw/` (または設定された `logging.file`) 下のローテーションするファイルログを監視してください。Docker外でmacOSアプリも実行する場合、そのサービスログは再び別個です: `~/.openclaw/logs/gateway.log`、`~/.openclaw/logs/gateway.err.log`、`/tmp/openclaw/openclaw-gateway.log`。

## エージェントサンドボックス (ホストゲートウェイ + Dockerツール)

詳細: [サンドボックス化](../gateway/sandboxing.md)

### 機能

`agents.defaults.sandbox` が有効な場合、**メイン以外のセッション**はツールをDockerコンテナ内で実行します。ゲートウェイはホスト上に残りますが、ツール実行は分離されます:

-   スコープ: デフォルトで `"agent"` (エージェントごとに1コンテナ + ワークスペース)
-   スコープ: セッションごとの分離には `"session"`
-   スコープごとのワークスペースフォルダが `/workspace` にマウント
-   オプションのエージェントワークスペースアクセス (`agents.defaults.sandbox.workspaceAccess`)
-   許可/拒否ツールポリシー (拒否が優先)
-   受信メディアはアクティブなサンドボックスワークスペース (`media/inbound/*`) にコピーされ、ツールが読み取れるようになります (`workspaceAccess: "rw"` の場合、これはエージェントワークスペースに配置されます)

警告: `scope: "shared"` はセッション間の分離を無効化します。すべてのセッションが1つのコンテナと1つのワークスペースを共有します。

### エージェントごとのサンドボックスプロファイル (マルチエージェント)

マルチエージェントルーティングを使用する場合、各エージェントはサンドボックス + ツール設定を上書きできます: `agents.list[].sandbox` および `agents.list[].tools` (さらに `agents.list[].tools.sandbox.tools`)。これにより、1つのゲートウェイで混合アクセスレベルを実行できます:

-   フルアクセス (個人エージェント)
-   読み取り専用ツール + 読み取り専用ワークスペース (家族/仕事エージェント)
-   ファイルシステム/シェルツールなし (公開エージェント)

例、優先順位、トラブルシューティングについては[マルチエージェントサンドボックス & ツール](../tools/multi-agent-sandbox-tools.md)を参照してください。

### デフォルトの動作

-   イメージ: `openclaw-sandbox:bookworm-slim`
-   エージェントごとに1コンテナ
-   エージェントワークスペースアクセス: `workspaceAccess: "none"` (デフォルト) は `~/.openclaw/sandboxes` を使用
    -   `"ro"` はサンドボックスワークスペースを `/workspace` に保持し、エージェントワークスペースを読み取り専用で `/agent` にマウント (`write`/`edit`/`apply_patch` を無効化)
    -   `"rw"` はエージェントワークスペースを読み書き可能で `/workspace` にマウント
-   自動プルーン: アイドル > 24時間 または 経過時間 > 7日
-   ネットワーク: デフォルトで `none` (エグレスが必要な場合は明示的にオプトイン)
    -   `host` はブロックされます。
    -   `container:` はデフォルトでブロックされます (名前空間結合リスク)。
-   デフォルト許可: `exec`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   デフォルト拒否: `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`

### サンドボックス化を有効化

`setupCommand` でパッケージをインストールする予定の場合、注意:

-   デフォルトの `docker.network` は `"none"` (エグレスなし)。
-   `docker.network: "host"` はブロ