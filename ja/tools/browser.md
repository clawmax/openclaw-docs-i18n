

  ブラウザ

  
# ブラウザ (OpenClaw マネージド)

OpenClaw は、エージェントが制御する**専用の Chrome/Brave/Edge/Chromium プロファイル**を実行できます。これはあなたの個人用ブラウザから隔離されており、ゲートウェイ内の小さなローカル制御サービス（ループバックのみ）を通じて管理されます。初心者向けの見方：

-   これを**分離された、エージェント専用のブラウザ**と考えてください。
-   `openclaw` プロファイルは、あなたの個人用ブラウザプロファイルに**触れません**。
-   エージェントは安全なレーン内で**タブを開き、ページを読み、クリックし、タイプ**できます。
-   デフォルトの `chrome` プロファイルは、拡張機能リレーを介して**システムデフォルトの Chromium ブラウザ**を使用します。隔離されたマネージドブラウザには `openclaw` に切り替えてください。

## 得られるもの

-   **openclaw** という名前の分離されたブラウザプロファイル（デフォルトでオレンジ色のアクセント）。
-   決定論的なタブ制御（リスト表示/開く/フォーカス/閉じる）。
-   エージェントアクション（クリック/タイプ/ドラッグ/選択）、スナップショット、スクリーンショット、PDF。
-   オプションのマルチプロファイルサポート（`openclaw`、`work`、`remote`、…）。

このブラウザは、あなたの日常的なブラウザ**ではありません**。エージェント自動化と検証のための安全で隔離された作業面です。

## クイックスタート

```bash
openclaw browser --browser-profile openclaw status
openclaw browser --browser-profile openclaw start
openclaw browser --browser-profile openclaw open https://example.com
openclaw browser --browser-profile openclaw snapshot
```

「Browser disabled」と表示された場合は、設定で有効にして（下記参照）ゲートウェイを再起動してください。

## プロファイル: openclaw 対 chrome

-   `openclaw`: マネージド、隔離されたブラウザ（拡張機能不要）。
-   `chrome`: **システムブラウザ**への拡張機能リレー（OpenClaw 拡張機能がタブにアタッチされている必要があります）。

デフォルトでマネージドモードを使用したい場合は、`browser.defaultProfile: "openclaw"` を設定します。

## 設定

ブラウザ設定は `~/.openclaw/openclaw.json` にあります。

```json
{
  browser: {
    enabled: true, // デフォルト: true
    ssrfPolicy: {
      dangerouslyAllowPrivateNetwork: true, // デフォルトの信頼ネットワークモード
      // allowPrivateNetwork: true, // 互換性のためのレガシーエイリアス
      // hostnameAllowlist: ["*.example.com", "example.com"],
      // allowedHostnames: ["localhost"],
    },
    // cdpUrl: "http://127.0.0.1:18792", // レガシー単一プロファイルオーバーライド
    remoteCdpTimeoutMs: 1500, // リモート CDP HTTP タイムアウト (ms)
    remoteCdpHandshakeTimeoutMs: 3000, // リモート CDP WebSocket ハンドシェイクタイムアウト (ms)
    defaultProfile: "chrome",
    color: "#FF4500",
    headless: false,
    noSandbox: false,
    attachOnly: false,
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser",
    profiles: {
      openclaw: { cdpPort: 18800, color: "#FF4500" },
      work: { cdpPort: 18801, color: "#0066CC" },
      remote: { cdpUrl: "http://10.0.0.42:9222", color: "#00AA00" },
    },
  },
}
```

注記:

-   ブラウザ制御サービスは、`gateway.port` から派生したポート（デフォルト: `18791`、これは gateway + 2）でループバックにバインドします。リレーは次のポート（`18792`）を使用します。
-   ゲートウェイポート（`gateway.port` または `OPENCLAW_GATEWAY_PORT`）をオーバーライドすると、派生するブラウザポートも同じ「ファミリー」に留まるようにシフトします。
-   `cdpUrl` は未設定の場合、リレーポートをデフォルトとします。
-   `remoteCdpTimeoutMs` は、リモート（非ループバック）CDP の到達可能性チェックに適用されます。
-   `remoteCdpHandshakeTimeoutMs` は、リモート CDP WebSocket の到達可能性チェックに適用されます。
-   ブラウザナビゲーション/タブオープンは、ナビゲーション前に SSRF ガードされ、ナビゲーション後の最終的な `http(s)` URL に対してベストエフォートで再チェックされます。
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork` はデフォルトで `true`（信頼ネットワークモデル）です。厳密なパブリック専用ブラウジングのためには `false` に設定してください。
-   `browser.ssrfPolicy.allowPrivateNetwork` は、互換性のためのレガシーエイリアスとして引き続きサポートされます。
-   `attachOnly: true` は「ローカルブラウザを起動しない。既に実行されている場合にのみアタッチする」ことを意味します。
-   `color` とプロファイルごとの `color` は、どのプロファイルがアクティブかわかるようにブラウザ UI を着色します。
-   デフォルトプロファイルは `openclaw`（OpenClaw マネージドスタンドアロンブラウザ）です。Chrome 拡張機能リレーを選択するには `defaultProfile: "chrome"` を使用します。
-   自動検出順序: Chromium ベースの場合はシステムデフォルトブラウザ、それ以外は Chrome → Brave → Edge → Chromium → Chrome Canary。
-   ローカルの `openclaw` プロファイルは `cdpPort`/`cdpUrl` を自動割り当てします — これらはリモート CDP の場合にのみ設定してください。

## Brave（または別の Chromium ベースブラウザ）を使用する

**システムデフォルト**のブラウザが Chromium ベース（Chrome/Brave/Edge など）の場合、OpenClaw は自動的にそれを使用します。自動検出をオーバーライドするには `browser.executablePath` を設定します: CLI 例:

```bash
openclaw config set browser.executablePath "/usr/bin/google-chrome"
```

```
// macOS
{
  browser: {
    executablePath: "/Applications/Brave Browser.app/Contents/MacOS/Brave Browser"
  }
}

// Windows
{
  browser: {
    executablePath: "C:\\Program Files\\BraveSoftware\\Brave-Browser\\Application\\brave.exe"
  }
}

// Linux
{
  browser: {
    executablePath: "/usr/bin/brave-browser"
  }
}
```

## ローカル制御とリモート制御

-   **ローカル制御 (デフォルト):** ゲートウェイがループバック制御サービスを起動し、ローカルブラウザを起動できます。
-   **リモート制御 (ノードホスト):** ブラウザがあるマシンでノードホストを実行します。ゲートウェイはブラウザアクションをそれにプロキシします。
-   **リモート CDP:** `browser.profiles..cdpUrl`（または `browser.cdpUrl`）を設定して、リモートの Chromium ベースブラウザにアタッチします。この場合、OpenClaw はローカルブラウザを起動しません。

リモート CDP URL には認証を含めることができます:

-   クエリトークン（例: `https://provider.example?token=`）
-   HTTP Basic 認証（例: `https://user:pass@provider.example`）

OpenClaw は、`/json/*` エンドポイントを呼び出すときや CDP WebSocket に接続するときに認証情報を保持します。トークンは設定ファイルにコミットするのではなく、環境変数やシークレットマネージャーを使用することを推奨します。

## ノードブラウザプロキシ（ゼロ設定デフォルト）

ブラウザがあるマシンで**ノードホスト**を実行する場合、OpenClaw は追加のブラウザ設定なしで、ブラウザツール呼び出しをそのノードに自動ルーティングできます。これはリモートゲートウェイのデフォルトパスです。注記:

-   ノードホストは、そのローカルブラウザ制御サーバーを**プロキシコマンド**経由で公開します。
-   プロファイルはノード自身の `browser.profiles` 設定（ローカルと同じ）から取得されます。
-   不要な場合は無効にできます:
    -   ノード側: `nodeHost.browserProxy.enabled=false`
    -   ゲートウェイ側: `gateway.nodes.browser.mode="off"`

## Browserless（ホストされたリモート CDP）

[Browserless](https://browserless.io) は、HTTPS 経由で CDP エンドポイントを公開するホストされた Chromium サービスです。OpenClaw ブラウザプロファイルを Browserless リージョンエンドポイントに向け、API キーで認証できます。例:

```json
{
  browser: {
    enabled: true,
    defaultProfile: "browserless",
    remoteCdpTimeoutMs: 2000,
    remoteCdpHandshakeTimeoutMs: 4000,
    profiles: {
      browserless: {
        cdpUrl: "https://production-sfo.browserless.io?token=<BROWSERLESS_API_KEY>",
        color: "#00AA00",
      },
    },
  },
}
```

注記:

-   `<BROWSERLESS_API_KEY>` を実際の Browserless トークンに置き換えてください。
-   Browserless アカウントに一致するリージョンエンドポイントを選択してください（ドキュメント参照）。

## セキュリティ

重要な考え方:

-   ブラウザ制御はループバックのみです。アクセスはゲートウェイの認証またはノードペアリングを経由します。
-   ブラウザ制御が有効で認証が設定されていない場合、OpenClaw は起動時に `gateway.auth.token` を自動生成し、設定に保存します。
-   ゲートウェイとノードホストはプライベートネットワーク（Tailscale）上に保ち、公開暴露を避けてください。
-   リモート CDP URL/トークンはシークレットとして扱い、環境変数やシークレットマネージャーを優先してください。

リモート CDP のヒント:

-   可能な限り HTTPS エンドポイントと短命トークンを優先してください。
-   設定ファイルに長寿命トークンを直接埋め込むのは避けてください。

## プロファイル（マルチブラウザ）

OpenClaw は複数の名前付きプロファイル（ルーティング設定）をサポートします。プロファイルは以下のいずれかです:

-   **openclaw マネージド**: 独自のユーザーデータディレクトリ + CDP ポートを持つ専用の Chromium ベースブラウザインスタンス
-   **リモート**: 明示的な CDP URL（別の場所で実行されている Chromium ベースブラウザ）
-   **拡張機能リレー**: ローカルリレー + Chrome 拡張機能を介した既存の Chrome タブ

デフォルト:

-   `openclaw` プロファイルは、存在しない場合に自動作成されます。
-   `chrome` プロファイルは、Chrome 拡張機能リレー用に組み込まれています（デフォルトで `http://127.0.0.1:18792` を指します）。
-   ローカル CDP ポートはデフォルトで **18800–18899** から割り当てられます。
-   プロファイルを削除すると、そのローカルデータディレクトリはゴミ箱に移動します。

すべての制御エンドポイントは `?profile=` を受け入れます。CLI は `--browser-profile` を使用します。

## Chrome 拡張機能リレー（既存の Chrome を使用）

OpenClaw は、ローカル CDP リレー + Chrome 拡張機能を介して**既存の Chrome タブ**（別の「openclaw」Chrome インスタンスなし）を駆動することもできます。完全ガイド: [Chrome 拡張機能](./chrome-extension.md) フロー:

-   ゲートウェイはローカル（同じマシン）で実行されるか、ノードホストがブラウザマシンで実行されます。
-   ローカル**リレーサーバー**がループバック `cdpUrl`（デフォルト: `http://127.0.0.1:18792`）でリッスンします。
-   タブにアタッチするために**OpenClaw Browser Relay**拡張機能アイコンをクリックします（自動アタッチはしません）。
-   エージェントは適切なプロファイルを選択することで、通常の `browser` ツールを介してそのタブを制御します。

ゲートウェイが別の場所で実行される場合は、ゲートウェイがブラウザアクションをプロキシできるように、ブラウザマシンでノードホストを実行します。

### サンドボックス化されたセッション

エージェントセッションがサンドボックス化されている場合、`browser` ツールはデフォルトで `target="sandbox"`（サンドボックスブラウザ）になる可能性があります。Chrome 拡張機能リレーの引き継ぎにはホストブラウザ制御が必要なので、以下のいずれかを行います:

-   セッションを非サンドボックスで実行する、または
-   `agents.defaults.sandbox.browser.allowHostControl: true` を設定し、ツールを呼び出すときに `target="host"` を使用します。

### セットアップ

1.  拡張機能をロード（開発/アンパック）:

```bash
openclaw browser extension install
```

-   Chrome → `chrome://extensions` → 「デベロッパーモード」を有効化
-   「パッケージ化されていない拡張機能を読み込む」 → `openclaw browser extension path` で出力されたディレクトリを選択
-   拡張機能をピン留めし、制御したいタブでクリック（バッジに `ON` と表示）。

2.  使用:

-   CLI: `openclaw browser --browser-profile chrome tabs`
-   エージェントツール: `browser` に `profile="chrome"` を指定

オプション: 別の名前やリレーポートが必要な場合は、独自のプロファイルを作成:

```bash
openclaw browser create-profile \
  --name my-chrome \
  --driver extension \
  --cdp-url http://127.0.0.1:18792 \
  --color "#00AA00"
```

注記:

-   このモードは、ほとんどの操作（スクリーンショット/スナップショット/アクション）に Playwright-on-CDP に依存します。
-   拡張機能アイコンをもう一度クリックしてデタッチします。

## 隔離保証

-   **専用ユーザーデータディレクトリ**: 個人用ブラウザプロファイルに触れることはありません。
-   **専用ポート**: 開発ワークフローとの衝突を防ぐため `9222` を避けます。
-   **決定論的タブ制御**: 「最後のタブ」ではなく `targetId` でターゲットタブを指定します。

## ブラウザ選択

ローカル起動時、OpenClaw は利用可能な最初のブラウザを選択します:

1.  Chrome
2.  Brave
3.  Edge
4.  Chromium
5.  Chrome Canary

`browser.executablePath` でオーバーライドできます。プラットフォーム:

-   macOS: `/Applications` と `~/Applications` をチェック。
-   Linux: `google-chrome`、`brave`、`microsoft-edge`、`chromium` などを検索。
-   Windows: 一般的なインストール場所をチェック。

## 制御 API（オプション）

ローカル統合のみのために、ゲートウェイは小さなループバック HTTP API を公開します:

-   ステータス/開始/停止: `GET /`、`POST /start`、`POST /stop`
-   タブ: `GET /tabs`、`POST /tabs/open`、`POST /tabs/focus`、`DELETE /tabs/:targetId`
-   スナップショット/スクリーンショット: `GET /snapshot`、`POST /screenshot`
-   アクション: `POST /navigate`、`POST /act`
-   フック: `POST /hooks/file-chooser`、`POST /hooks/dialog`
-   ダウンロード: `POST /download`、`POST /wait/download`
-   デバッグ: `GET /console`、`POST /pdf`
-   デバッグ: `GET /errors`、`GET /requests`、`POST /trace/start`、`POST /trace/stop`、`POST /highlight`
-   ネットワーク: `POST /response/body`
-   状態: `GET /cookies`、`POST /cookies/set`、`POST /cookies/clear`
-   状態: `GET /storage/:kind`、`POST /storage/:kind/set`、`POST /storage/:kind/clear`
-   設定: `POST /set/offline`、`POST /set/headers`、`POST /set/credentials`、`POST /set/geolocation`、`POST /set/media`、`POST /set/timezone`、`POST /set/locale`、`POST /set/device`

すべてのエンドポイントは `?profile=` を受け入れます。ゲートウェイ認証が設定されている場合、ブラウザ HTTP ルートも認証を要求します:

-   `Authorization: Bearer `
-   `x-openclaw-password: ` またはそのパスワードを使用した HTTP Basic 認証

### Playwright 要件

一部の機能（ナビゲート/アクション/AI スナップショット/ロールスナップショット、要素スクリーンショット、PDF）には Playwright が必要です。Playwright がインストールされていない場合、それらのエンドポイントは明確な 501 エラーを返します。ARIA スナップショットと基本的なスクリーンショットは、openclaw マネージド Chrome では引き続き動作します。Chrome 拡張機能リレードライバーの場合、ARIA スナップショットとスクリーンショットには Playwright が必要です。`Playwright is not available in this gateway build` が表示された場合は、完全な Playwright パッケージ（`playwright-core` ではない）をインストールしてゲートウェイを再起動するか、ブラウザサポート付きで OpenClaw を再インストールしてください。

#### Docker Playwright インストール

ゲートウェイが Docker で実行されている場合、`npx playwright`（npm オーバーライド競合）は避けてください。代わりにバンドルされた CLI を使用します:

```bash
docker compose run --rm openclaw-cli \
  node /app/node_modules/playwright-core/cli.js install chromium
```

ブラウザダウンロードを永続化するには、`PLAYWRIGHT_BROWSERS_PATH`（例: `/home/node/.cache/ms-playwright`）を設定し、`/home/node` が `OPENCLAW_HOME_VOLUME` またはバインドマウントを介して永続化されていることを確認してください。[Docker](../install/docker.md) を参照。

## 仕組み（内部）

高レベルフロー:

-   小さな**制御サーバー**が HTTP リクエストを受け付けます。
-   **CDP** を介して Chromium ベースブラウザ（Chrome/Brave/Edge/Chromium）に接続します。
-   高度なアクション（クリック/タイプ/スナップショット/PDF）には、CDP 上で **Playwright** を使用します。
-   Playwright がない場合、非 Playwright 操作のみが利用可能です。

この設計により、エージェントは安定した決定論的インターフェース上に保ちながら、ローカル/リモートブラウザやプロファイルを交換できます。

## CLI クイックリファレンス

すべてのコマンドは `--browser-profile ` を受け入れて特定のプロファイルをターゲットにします。すべてのコマンドは、機械可読な出力（安定したペイロード）のために `--json` も受け入れます。基本:

-   `openclaw browser status`
-   `openclaw browser start`
-   `openclaw browser stop`
-   `openclaw browser tabs`
-   `openclaw browser tab`
-   `openclaw browser tab new`
-   `openclaw browser tab select 2`
-   `openclaw browser tab close 2`
-   `openclaw browser open https://example.com`
-   `openclaw browser focus abcd1234`
-   `openclaw browser close abcd1234`

検査:

-   `openclaw browser screenshot`
-   `openclaw browser screenshot --full-page`
-   `openclaw browser screenshot --ref 12`
-   `openclaw browser screenshot --ref e12`
-   `openclaw browser snapshot`
-   `openclaw browser snapshot --format aria --limit 200`
-   `openclaw browser snapshot --interactive --compact --depth 6`
-   `openclaw browser snapshot --efficient`
-   `openclaw browser snapshot --labels`
-   `openclaw browser snapshot --selector "#main" --interactive`
-   `openclaw browser snapshot --frame "iframe#main" --interactive`
-   `openclaw browser console --level error`
-   `openclaw browser errors --clear`
-   `openclaw browser requests --filter api --clear`
-   `openclaw browser pdf`
-   `openclaw browser responsebody "**/api" --max-chars 5000`

アクション:

-   `openclaw browser navigate https://example.com`
-   `openclaw browser resize 1280 720`
-   `openclaw browser click 12 --double`
-   `openclaw browser click e12 --double`
-   `openclaw browser type 23 "hello" --submit`
-   `openclaw browser press Enter`
-   `openclaw browser hover 44`
-   `openclaw browser scrollintoview e12`
-   `openclaw browser drag 10 11`
-   `openclaw browser select 9 OptionA OptionB`
-   `openclaw browser download e12 report.pdf`
-   `openclaw browser waitfordownload report.pdf`
-   `openclaw browser upload /tmp/openclaw/uploads/file.pdf`
-   `openclaw browser fill --fields '[{"ref":"1","type":"text","value":"Ada"}]'`
-   `openclaw browser dialog --accept`
-   `openclaw browser wait --text "Done"`
-   `openclaw browser wait "#main" --url "**/dash" --load networkidle --fn "window.ready===true"`
-   `openclaw browser evaluate --fn '(el) => el.textContent' --ref 7`
-   `openclaw browser highlight e12`
-   `openclaw browser trace start`
-   `openclaw browser trace stop`

状態:

-   `openclaw browser cookies`
-   `openclaw browser cookies set session abc123 --url "https://example.com"`
-   `openclaw browser cookies clear`
-   `openclaw browser storage local get`
-   `openclaw browser storage local set theme dark`
-   `openclaw browser storage session clear`
-   `openclaw browser set offline on`
-   `openclaw browser set headers --headers-json '{"X-Debug":"1"}'`
-   `openclaw browser set credentials user pass`
-   `openclaw browser set credentials --clear`
-   `openclaw browser set geo 37.7749 -122.4194 --origin "https://example.com"`
-   `openclaw browser set geo --clear`
-   `openclaw browser set media dark`
-   `openclaw browser set timezone America/New_York`
-   `openclaw browser set locale en-US`
-   `openclaw browser set device "iPhone 14"`

注記:

-   `upload` と `dialog` は**アーミング**呼び出しです。ファイル選択/ダイアログをトリガーするクリック/押下の前に実行してください。
-   ダウンロードとトレース出力パスは OpenClaw 一時ルートに制限されます:
    -   トレース: `/tmp/openclaw`（フォールバック: `${os.tmpdir()}/openclaw`）
    -   ダウンロード: `/tmp/openclaw/downloads`（フォールバック: `${os.tmpdir()}/openclaw/downloads`）
-   アップロードパスは OpenClaw 一時アップロードルートに制限されます:
    -   アップロード: `/tmp/openclaw/uploads`（フォールバック: `${os.tmpdir()}/openclaw/uploads`）
-   `upload` は `--input-ref` または `--element` を介してファイル入力を直接設定することもできます。
-   `snapshot`:
    -   `--format ai`（Playwright インストール時のデフォルト）: 数値 ref（`aria-ref=""`）を含む AI スナップショットを返します。
    -   `--format aria`: アクセシビリティツリーを返します（ref なし、検査のみ）。
    -   `--efficient`（または `--mode efficient`）: コンパクトなロールスナップショットプリセット（interactive + compact + depth + 低い maxChars）。
    -   設定デフォルト（ツール/CLI のみ）: 呼び出し元がモードを渡さない場合に効率的なスナップショットを使用するには、`browser.snapshotDefaults.mode: "efficient"` を設定します（[ゲートウェイ設定](../gateway/configuration.md#browser-openclaw-managed-browser) 参照）。
    -   ロールスナップショットオプション（`--interactive`、`--compact`、`--depth`、`--selector`）は、`ref=e12` のような ref を持つロールベースのスナップショットを強制します。
    -   `--frame ""` は、ロールスナップショットを iframe にスコープします（`e12` のようなロール ref とペアになります）。
    -   `--interactive` は、インタラクティブ要素のフラットで選択しやすいリストを出力します（アクション駆動に最適）。
    -   `--labels` は、ref ラベルがオーバーレイされたビューポートのみのスクリーンショットを追加します（`MEDIA:` を出力）。
-   `click`/`type`/などは、`snapshot` からの `ref`（数値 `12` またはロール ref `e12`）を必要とします。アクションのために CSS セレクターは意図的にサポートされていません。

## スナップショットと ref

OpenClaw は 2 つの「スナップショット」スタイルをサポートします:

-   **AI スナップショット (数値 ref)**: `openclaw browser snapshot`（デフォルト; `--format ai`）
    -   出力: 数値 ref を含むテキストスナップショット。
    -   アクション: `openclaw browser click 12`、`openclaw browser type 23 "hello"`。
    -   内部的に、ref は Playwright の `aria-ref` を介して解決されます。
-   **ロールスナップショット (`e12` のようなロール ref)**: `openclaw browser snapshot --interactive`（または `--compact`、`--depth`、`--selector`、`--frame`）
    -   出力: `[ref=e12]`（およびオプションの `[nth=1]`）を含むロールベースのリスト/ツリー。
    -   アクション: `openclaw browser click e12`、`openclaw browser highlight e12`。
    -   内部的に、ref は `getByRole(...)`（重複には `nth()` を追加）を介して解決されます。
    -   `--labels` を追加して、`e12` ラベルがオーバーレイされたビューポートスクリーンショットを含めます。

Ref の動作:

-   Ref は**ナビゲーション間で安定していません**。何か失敗した場合は、`snapshot` を再実行して新しい ref を使用してください。
-   ロールスナップショットが `--frame` で取得された場合、ロール ref は次のロールスナップショットまでその iframe にスコープされます。

## 待機パワーアップ

時間/テキスト以外でも待機できます:

-   URL を待機（Playwright でサポートされるグロブ）:
    -   `openclaw browser wait --url "**/dash"`
-   ロード状態を待機:
    -   `openclaw browser wait --load networkidle`
-   JS 述語を待機:
    -   `openclaw browser wait --fn "window.ready===true"`
-   セレクターが表示されるのを待機:
    -   `openclaw browser wait "#main"`

これらを組み合わせることができます:

```bash
openclaw browser wait "#main" \
  --url "**/dash" \
  --load networkidle \
  --fn "window.ready===true" \
  --timeout-ms 15000
```

## デバッグワークフロー

アクションが失敗した場合（例: 「not visible」、「strict mode violation」、「covered」）:

1.  `openclaw browser snapshot --interactive`
2.  `click ` / `type ` を使用（インタラクティブモードではロール ref を優先）
3.  それでも失敗する場合: `openclaw browser highlight ` で Playwright がターゲットにしているものを確認
4.  ページが奇妙な動作をする場合:
    -   `openclaw browser errors --clear`
    -   `openclaw browser requests --filter api --clear`
5.  詳細なデバッグ: トレースを記録:
    -   `openclaw browser trace start`
    -   問題を再現
    -   `openclaw browser trace stop`（`TRACE:` を出力）

## JSON 出力

`--json` はスクリプトと構造化ツール用です。例:

```bash
openclaw browser status --json
openclaw browser snapshot --interactive --json
openclaw browser requests --filter api --json
openclaw browser cookies --json
```

JSON でのロールスナップショットには、`refs` に加えて小さな `stats` ブロック（行数/文字数/ref 数/インタラクティブ）が含まれ、ツールがペイロードサイズと密度を推論できるようになります。

## 状態と環境ノブ

これらは「サイトを X のように動作させる」ワークフローに役立ちます:

-   クッキー: `cookies`、`cookies set`、`cookies clear`
-   ストレージ: `storage local|session get|set|clear`
-   オフライン: `set offline on|off`
-   ヘッダー: `set headers --headers-json '{"X-Debug":"1"}'`（レガシー `set headers --json '{"X-Debug":"1"}'` も引き続きサポート）
-   HTTP 基本認証: `set credentials user pass`（または `--clear`）
-   地理位置情報: `set geo   --origin "https://example.com"`（または `--clear`）
-   メディア: `set media dark|light|no-preference|none`
-   タイムゾーン / ロケール: `set timezone ...`、`set locale ...`
-   デバイス / ビューポ