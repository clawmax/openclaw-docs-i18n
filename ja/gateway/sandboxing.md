

  セキュリティとサンドボックス化

  
# サンドボックス化

OpenClawは、**ツールをDockerコンテナ内で実行**し、被害範囲を縮小することができます。これは**オプション**機能であり、設定（`agents.defaults.sandbox` または `agents.list[].sandbox`）によって制御されます。サンドボックス化がオフの場合、ツールはホスト上で実行されます。ゲートウェイはホスト上に残り、有効時にはツール実行が隔離されたサンドボックス内で行われます。これは完全なセキュリティ境界ではありませんが、モデルが愚かなことをした際のファイルシステムやプロセスへのアクセスを実質的に制限します。

## サンドボックス化されるもの

-   ツール実行（`exec`、`read`、`write`、`edit`、`apply_patch`、`process` など）。
-   オプションのサンドボックス化ブラウザ（`agents.defaults.sandbox.browser`）。
    -   デフォルトでは、ブラウザツールが必要とする際にサンドボックスブラウザは自動起動します（CDPが到達可能であることを保証）。`agents.defaults.sandbox.browser.autoStart` と `agents.defaults.sandbox.browser.autoStartTimeoutMs` で設定します。
    -   デフォルトでは、サンドボックスブラウザコンテナはグローバルな `bridge` ネットワークの代わりに専用のDockerネットワーク（`openclaw-sandbox-browser`）を使用します。`agents.defaults.sandbox.browser.network` で設定します。
    -   オプションの `agents.defaults.sandbox.browser.cdpSourceRange` は、CIDR許可リスト（例：`172.21.0.1/32`）でコンテナエッジへのCDPインバウンドを制限します。
    -   noVNCオブザーバーアクセスはデフォルトでパスワード保護されています。OpenClawは短命のトークンURLを生成し、ローカルのブートストラップページを提供し、パスワードをURLフラグメントに含めてnoVNCを開きます（クエリ/ヘッダーログには含まれません）。
    -   `agents.defaults.sandbox.browser.allowHostControl` は、サンドボックス化されたセッションが明示的にホストブラウザをターゲットにすることを許可します。
    -   オプションの許可リストが `target: "custom"` を制御します：`allowedControlUrls`、`allowedControlHosts`、`allowedControlPorts`。

サンドボックス化されないもの:

-   ゲートウェイプロセス自体。
-   ホスト上での実行が明示的に許可されたツール（例：`tools.elevated`）。
    -   **昇格されたexecはホスト上で実行され、サンドボックス化をバイパスします。**
    -   サンドボックス化がオフの場合、`tools.elevated` は実行を変更しません（すでにホスト上）。[昇格モード](../tools/elevated.md)を参照してください。

## モード

`agents.defaults.sandbox.mode` は、**いつ**サンドボックス化が使用されるかを制御します：

-   `"off"`: サンドボックス化なし。
-   `"non-main"`: **非メイン**セッションのみをサンドボックス化（通常のチャットをホストで行いたい場合のデフォルト）。
-   `"all"`: すべてのセッションがサンドボックス内で実行されます。注：`"non-main"` はエージェントIDではなく、`session.mainKey`（デフォルト `"main"`）に基づきます。グループ/チャネルセッションは独自のキーを使用するため、非メインとしてカウントされ、サンドボックス化されます。

## スコープ

`agents.defaults.sandbox.scope` は、**いくつのコンテナ**が作成されるかを制御します：

-   `"session"` (デフォルト): セッションごとに1つのコンテナ。
-   `"agent"`: エージェントごとに1つのコンテナ。
-   `"shared"`: すべてのサンドボックス化セッションで共有される1つのコンテナ。

## ワークスペースアクセス

`agents.defaults.sandbox.workspaceAccess` は、**サンドボックスが何を見ることができるか**を制御します：

-   `"none"` (デフォルト): ツールは `~/.openclaw/sandboxes` 下のサンドボックスワークスペースを見ます。
-   `"ro"`: エージェントワークスペースを読み取り専用で `/agent` にマウントします（`write`/`edit`/`apply_patch` を無効化）。
-   `"rw"`: エージェントワークスペースを読み書き可能で `/workspace` にマウントします。

インバウンドメディアはアクティブなサンドボックスワークスペース（`media/inbound/*`）にコピーされます。スキルに関する注意：`read` ツールはサンドボックスルートに基づきます。`workspaceAccess: "none"` の場合、OpenClawは適格なスキルをサンドボックスワークスペース（`.../skills`）にミラーリングして読み取り可能にします。`"rw"` の場合、ワークスペーススキルは `/workspace/skills` から読み取り可能です。

## カスタムバインドマウント

`agents.defaults.sandbox.docker.binds` は、追加のホストディレクトリをコンテナ内にマウントします。形式：`host:container:mode`（例：`"/home/user/source:/source:rw"`）。グローバルおよびエージェントごとのバインドは**マージ**されます（置き換えられません）。`scope: "shared"` の場合、エージェントごとのバインドは無視されます。`agents.defaults.sandbox.browser.binds` は、追加のホストディレクトリを**サンドボックスブラウザ**コンテナのみにマウントします。

-   設定された場合（`[]` を含む）、ブラウザコンテナに対して `agents.defaults.sandbox.docker.binds` を置き換えます。
-   省略された場合、ブラウザコンテナは `agents.defaults.sandbox.docker.binds` にフォールバックします（後方互換性）。

例（読み取り専用ソース + 追加データディレクトリ）：

```json
{
  agents: {
    defaults: {
      sandbox: {
        docker: {
          binds: ["/home/user/source:/source:ro", "/var/data/myapp:/data:ro"],
        },
      },
    },
    list: [
      {
        id: "build",
        sandbox: {
          docker: {
            binds: ["/mnt/cache:/cache:rw"],
          },
        },
      },
    ],
  },
}
```

セキュリティ上の注意：

-   バインドはサンドボックスファイルシステムをバイパスします：設定したモード（`:ro` または `:rw`）でホストパスを公開します。
-   OpenClawは危険なバインドソースをブロックします（例：`docker.sock`、`/etc`、`/proc`、`/sys`、`/dev`、およびそれらを公開する親マウント）。
-   機密性の高いマウント（シークレット、SSHキー、サービス認証情報）は、絶対に必要な場合を除き `:ro` にすべきです。
-   ワークスペースへの読み取りアクセスのみが必要な場合は、`workspaceAccess: "ro"` と組み合わせてください。バインドモードは独立したままです。
-   バインドがツールポリシーおよび昇格されたexecとどのように相互作用するかについては、[サンドボックス vs ツールポリシー vs 昇格](./sandbox-vs-tool-policy-vs-elevated.md)を参照してください。

## イメージ + セットアップ

デフォルトイメージ：`openclaw-sandbox:bookworm-slim` 一度ビルドします：

```
scripts/sandbox-setup.sh
```

注：デフォルトイメージには**Nodeは含まれていません**。スキルがNode（または他のランタイム）を必要とする場合、カスタムイメージを作成するか、`sandbox.docker.setupCommand` 経由でインストールしてください（ネットワークエグレス + 書き込み可能なルート + rootユーザーが必要）。より機能的なサンドボックスイメージ（例：`curl`、`jq`、`nodejs`、`python3`、`git` などの一般的なツールを含む）が必要な場合は、以下をビルドします：

```
scripts/sandbox-common-setup.sh
```

その後、`agents.defaults.sandbox.docker.image` を `openclaw-sandbox-common:bookworm-slim` に設定します。サンドボックス化ブラウザイメージ：

```
scripts/sandbox-browser-setup.sh
```

デフォルトでは、サンドボックスコンテナは**ネットワークなし**で実行されます。`agents.defaults.sandbox.docker.network` でオーバーライドします。バンドルされたサンドボックスブラウザイメージは、コンテナ化ワークロード向けに控えめなChromium起動デフォルトも適用します。現在のコンテナデフォルトには以下が含まれます：

-   `--remote-debugging-address=127.0.0.1`
-   `--remote-debugging-port=`
-   `--user-data-dir=${HOME}/.chrome`
-   `--no-first-run`
-   `--no-default-browser-check`
-   `--disable-3d-apis`
-   `--disable-gpu`
-   `--disable-dev-shm-usage`
-   `--disable-background-networking`
-   `--disable-extensions`
-   `--disable-features=TranslateUI`
-   `--disable-breakpad`
-   `--disable-crash-reporter`
-   `--disable-software-rasterizer`
-   `--no-zygote`
-   `--metrics-recording-only`
-   `--renderer-process-limit=2`
-   `--no-sandbox` および `--disable-setuid-sandbox` は `noSandbox` が有効な場合に適用されます。
-   3つのグラフィックス強化フラグ（`--disable-3d-apis`、`--disable-software-rasterizer`、`--disable-gpu`）はオプションであり、コンテナがGPUサポートを欠く場合に有用です。ワークロードがWebGLや他の3D/ブラウザ機能を必要とする場合は、`OPENCLAW_BROWSER_DISABLE_GRAPHICS_FLAGS=0` を設定してください。
-   `--disable-extensions` はデフォルトで有効化されており、拡張機能に依存するフローの場合は `OPENCLAW_BROWSER_DISABLE_EXTENSIONS=0` で無効化できます。
-   `--renderer-process-limit=2` は `OPENCLAW_BROWSER_RENDERER_PROCESS_LIMIT=` で制御され、`0` はChromiumのデフォルトを維持します。

異なるランタイムプロファイルが必要な場合は、カスタムブラウザイメージを使用し、独自のエントリーポイントを提供してください。ローカル（非コンテナ）のChromiumプロファイルの場合は、`browser.extraArgs` を使用して追加の起動フラグを追加します。セキュリティデフォルト：

-   `network: "host"` はブロックされます。
-   `network: "container:"` はデフォルトでブロックされます（名前空間結合バイパスリスク）。
-   非常時オーバーライド：`agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin: true`。

Dockerインストールとコンテナ化ゲートウェイの詳細はこちら：[Docker](../install/docker.md) Dockerゲートウェイデプロイメントの場合、`docker-setup.sh` がサンドボックス設定をブートストラップできます。`OPENCLAW_SANDBOX=1`（または `true`/`yes`/`on`）を設定してそのパスを有効にします。ソケットの場所は `OPENCLAW_DOCKER_SOCKET` でオーバーライドできます。完全なセットアップと環境変数リファレンス：[Docker](../install/docker.md#enable-agent-sandbox-for-docker-gateway-opt-in)。

## setupCommand（ワンタイムコンテナセットアップ）

`setupCommand` は、サンドボックスコンテナが作成された後（毎回の実行時ではなく）**一度だけ**実行されます。`sh -lc` 経由でコンテナ内で実行されます。パス：

-   グローバル：`agents.defaults.sandbox.docker.setupCommand`
-   エージェントごと：`agents.list[].sandbox.docker.setupCommand`

よくある落とし穴：

-   デフォルトの `docker.network` は `"none"`（エグレスなし）であるため、パッケージインストールは失敗します。
-   `docker.network: "container:"` は `dangerouslyAllowContainerNamespaceJoin: true` を必要とし、非常時のみの使用です。
-   `readOnlyRoot: true` は書き込みを防ぎます。`readOnlyRoot: false` を設定するか、カスタムイメージを作成してください。
-   パッケージインストールには `user` がrootである必要があります（`user` を省略するか `user: "0:0"` を設定）。
-   サンドボックスexecはホストの `process.env` を**継承しません**。スキルAPIキーには `agents.defaults.sandbox.docker.env`（またはカスタムイメージ）を使用してください。

## ツールポリシー + エスケープハッチ

ツールの許可/拒否ポリシーは、サンドボックスルールの前に適用されます。ツールがグローバルまたはエージェントごとに拒否されている場合、サンドボックス化によって復活することはありません。`tools.elevated` は、ホスト上で `exec` を実行する明示的なエスケープハッチです。`/exec` ディレクティブは認証された送信者のみに適用され、セッションごとに永続化されます。`exec` を強制的に無効化するには、ツールポリシー拒否を使用してください（[サンドボックス vs ツールポリシー vs 昇格](./sandbox-vs-tool-policy-vs-elevated.md)を参照）。デバッグ：

-   `openclaw sandbox explain` を使用して、有効なサンドボックスモード、ツールポリシー、修正設定キーを検査します。
-   「なぜこれがブロックされるのか？」というメンタルモデルについては、[サンドボックス vs ツールポリシー vs 昇格](./sandbox-vs-tool-policy-vs-elevated.md)を参照してください。ロックダウンを維持してください。

## マルチエージェントオーバーライド

各エージェントはサンドボックスとツールをオーバーライドできます：`agents.list[].sandbox` と `agents.list[].tools`（サンドボックスのツールポリシーには `agents.list[].tools.sandbox.tools` も）。優先順位については[マルチエージェントサンドボックス & ツール](../tools/multi-agent-sandbox-tools.md)を参照してください。

## 最小限の有効化例

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        scope: "session",
        workspaceAccess: "none",
      },
    },
  },
}
```

## 関連ドキュメント

-   [サンドボックス設定](./configuration.md#agentsdefaults-sandbox)
-   [マルチエージェントサンドボックス & ツール](../tools/multi-agent-sandbox-tools.md)
-   [セキュリティ](./security.md)

[セキュリティ](./security.md)[サンドボックス vs ツールポリシー vs 昇格](./sandbox-vs-tool-policy-vs-elevated.md)