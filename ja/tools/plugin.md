title: "OpenClaw スキルプラグインガイド インストール・設定・SDK"
description: "OpenClaw用プラグインのインストール、設定、開発方法を学びましょう。公式プラグイン、HTTPルート、ランタイムヘルパーで機能を拡張します。"
keywords: ["openclaw プラグイン", "スキルプラグイン", "プラグインインストール", "プラグイン sdk", "ゲートウェイ rpc", "エージェントツール", "プラグインマニフェスト", "ランタイムヘルパー"]
---

  スキル

  
# プラグイン

## クイックスタート (プラグイン初心者向け)

プラグインは、OpenClawに追加機能（コマンド、ツール、Gateway RPC）を提供する**小さなコードモジュール**です。多くの場合、コアのOpenClawにまだ組み込まれていない機能が必要なとき（またはオプション機能をメインインストールから分離したいとき）にプラグインを使用します。最短パス:

1.  既に読み込まれているものを確認:

```bash
openclaw plugins list
```

2.  公式プラグインをインストール (例: Voice Call):

```bash
openclaw plugins install @openclaw/voice-call
```

Npm仕様は**レジストリ専用**です（パッケージ名 + オプションで**正確なバージョン**または**dist-tag**）。Git/URL/ファイル仕様とsemver範囲は拒否されます。ベア仕様と`@latest`は安定版トラックに留まります。npmがこれらのいずれかをプレリリースに解決した場合、OpenClawは停止し、`@beta`/`@rc`などのプレリリースタグまたは正確なプレリリースバージョンを明示的にオプトインするよう要求します。

3.  Gatewayを再起動し、`plugins.entries..config`で設定します。

具体的なプラグインの例については[Voice Call](../plugins/voice-call.md)を参照してください。サードパーティのリストをお探しですか？[コミュニティプラグイン](../plugins/community.md)を参照してください。

## 利用可能なプラグイン (公式)

-   Microsoft Teamsは2026.1.15時点でプラグイン専用です。Teamsを使用する場合は`@openclaw/msteams`をインストールしてください。
-   Memory (Core) — バンドルされたメモリ検索プラグイン（`plugins.slots.memory`経由でデフォルト有効）
-   Memory (LanceDB) — バンドルされた長期メモリプラグイン（自動想起/キャプチャ；`plugins.slots.memory = "memory-lancedb"`を設定）
-   [Voice Call](../plugins/voice-call.md) — `@openclaw/voice-call`
-   [Zalo Personal](../plugins/zalouser.md) — `@openclaw/zalouser`
-   [Matrix](../channels/matrix.md) — `@openclaw/matrix`
-   [Nostr](../channels/nostr.md) — `@openclaw/nostr`
-   [Zalo](../channels/zalo.md) — `@openclaw/zalo`
-   [Microsoft Teams](../channels/msteams.md) — `@openclaw/msteams`
-   Google Antigravity OAuth (プロバイダー認証) — `google-antigravity-auth`としてバンドル（デフォルト無効）
-   Gemini CLI OAuth (プロバイダー認証) — `google-gemini-cli-auth`としてバンドル（デフォルト無効）
-   Qwen OAuth (プロバイダー認証) — `qwen-portal-auth`としてバンドル（デフォルト無効）
-   Copilot Proxy (プロバイダー認証) — ローカルVS Code Copilot Proxyブリッジ；組み込みの`github-copilot`デバイスログインとは異なる（バンドル、デフォルト無効）

OpenClawプラグインは、jiti経由でランタイムに読み込まれる**TypeScriptモジュール**です。**設定検証ではプラグインコードは実行されません**；代わりにプラグインマニフェストとJSON Schemaが使用されます。詳細は[プラグインマニフェスト](../plugins/manifest.md)を参照してください。プラグインは以下を登録できます:

-   Gateway RPCメソッド
-   Gateway HTTPルート
-   エージェントツール
-   CLIコマンド
-   バックグラウンドサービス
-   コンテキストエンジン
-   オプションの設定検証
-   **スキル** (プラグインマニフェストで`skills`ディレクトリをリストすることで)
-   **自動返信コマンド** (AIエージェントを起動せずに実行)

プラグインはGatewayと**同一プロセス内**で実行されるため、信頼できるコードとして扱ってください。ツール作成ガイド: [プラグインエージェントツール](../plugins/agent-tools.md)。

## ランタイムヘルパー

プラグインは`api.runtime`経由で選択されたコアヘルパーにアクセスできます。電話向けTTSの場合:

```typescript
const result = await api.runtime.tts.textToSpeechTelephony({
  text: "Hello from OpenClaw",
  cfg: api.config,
});
```

注意点:

-   コアの`messages.tts`設定（OpenAIまたはElevenLabs）を使用します。
-   PCMオーディオバッファ + サンプルレートを返します。プラグインはプロバイダー向けにリサンプリング/エンコードする必要があります。
-   電話向けのEdge TTSはサポートされていません。

STT/文字起こしの場合、プラグインは以下を呼び出せます:

```typescript
const { text } = await api.runtime.stt.transcribeAudioFile({
  filePath: "/tmp/inbound-audio.ogg",
  cfg: api.config,
  // MIMEタイプが確実に推測できない場合にオプション:
  mime: "audio/ogg",
});
```

注意点:

-   コアのメディア理解オーディオ設定（`tools.media.audio`）とプロバイダーフォールバック順序を使用します。
-   文字起こし出力が生成されない場合（例: スキップ/サポートされていない入力）、`{ text: undefined }`を返します。

## Gateway HTTPルート

プラグインは`api.registerHttpRoute(...)`でHTTPエンドポイントを公開できます。

```
api.registerHttpRoute({
  path: "/acme/webhook",
  auth: "plugin",
  match: "exact",
  handler: async (_req, res) => {
    res.statusCode = 200;
    res.end("ok");
    return true;
  },
});
```

ルートフィールド:

-   `path`: gateway HTTPサーバー下のルートパス。
-   `auth`: 必須。通常のgateway認証を要求するには`"gateway"`を、プラグイン管理の認証/Webhook検証には`"plugin"`を使用します。
-   `match`: オプション。`"exact"`（デフォルト）または`"prefix"`。
-   `replaceExisting`: オプション。同じプラグインが自身の既存のルート登録を置き換えることを許可します。
-   `handler`: ルートがリクエストを処理したときに`true`を返します。

注意点:

-   `api.registerHttpHandler(...)`は廃止されました。`api.registerHttpRoute(...)`を使用してください。
-   プラグインルートは明示的に`auth`を宣言する必要があります。
-   正確な`path + match`の競合は`replaceExisting: true`でない限り拒否され、あるプラグインが別のプラグインのルートを置き換えることはできません。
-   異なる`auth`レベルを持つ重複するルートは拒否されます。同じ認証レベルでのみ`exact`/`prefix`フォールスルーチェーンを維持してください。

## プラグインSDKインポートパス

プラグイン作成時には、モノリシックな`openclaw/plugin-sdk`インポートの代わりにSDKサブパスを使用してください:

-   `openclaw/plugin-sdk/core` 汎用プラグインAPI、プロバイダー認証タイプ、共有ヘルパー用。
-   `openclaw/plugin-sdk/compat` `core`よりも広範な共有ランタイムヘルパーを必要とするバンドル/内部プラグインコード用。
-   `openclaw/plugin-sdk/telegram` Telegramチャネルプラグイン用。
-   `openclaw/plugin-sdk/discord` Discordチャネルプラグイン用。
-   `openclaw/plugin-sdk/slack` Slackチャネルプラグイン用。
-   `openclaw/plugin-sdk/signal` Signalチャネルプラグイン用。
-   `openclaw/plugin-sdk/imessage` iMessageチャネルプラグイン用。
-   `openclaw/plugin-sdk/whatsapp` WhatsAppチャネルプラグイン用。
-   `openclaw/plugin-sdk/line` LINEチャネルプラグイン用。
-   `openclaw/plugin-sdk/msteams` バンドルされたMicrosoft Teamsプラグインサーフェス用。
-   バンドルされた拡張機能固有のサブパスも利用可能です: `openclaw/plugin-sdk/acpx`, `openclaw/plugin-sdk/bluebubbles`, `openclaw/plugin-sdk/copilot-proxy`, `openclaw/plugin-sdk/device-pair`, `openclaw/plugin-sdk/diagnostics-otel`, `openclaw/plugin-sdk/diffs`, `openclaw/plugin-sdk/feishu`, `openclaw/plugin-sdk/google-gemini-cli-auth`, `openclaw/plugin-sdk/googlechat`, `openclaw/plugin-sdk/irc`, `openclaw/plugin-sdk/llm-task`, `openclaw/plugin-sdk/lobster`, `openclaw/plugin-sdk/matrix`, `openclaw/plugin-sdk/mattermost`, `openclaw/plugin-sdk/memory-core`, `openclaw/plugin-sdk/memory-lancedb`, `openclaw/plugin-sdk/minimax-portal-auth`, `openclaw/plugin-sdk/nextcloud-talk`, `openclaw/plugin-sdk/nostr`, `openclaw/plugin-sdk/open-prose`, `openclaw/plugin-sdk/phone-control`, `openclaw/plugin-sdk/qwen-portal-auth`, `openclaw/plugin-sdk/synology-chat`, `openclaw/plugin-sdk/talk-voice`, `openclaw/plugin-sdk/test-utils`, `openclaw/plugin-sdk/thread-ownership`, `openclaw/plugin-sdk/tlon`, `openclaw/plugin-sdk/twitch`, `openclaw/plugin-sdk/voice-call`, `openclaw/plugin-sdk/zalo`, `openclaw/plugin-sdk/zalouser`.

互換性に関する注意:

-   `openclaw/plugin-sdk`は既存の外部プラグインに対して引き続きサポートされます。
-   新規および移行済みのバンドルプラグインは、チャネルまたは拡張機能固有のサブパスを使用してください。汎用サーフェスには`core`を、より広範な共有ヘルパーが必要な場合にのみ`compat`を使用してください。

## 読み取り専用チャネル検査

プラグインがチャネルを登録する場合、`resolveAccount(...)`と共に`plugin.config.inspectAccount(cfg, accountId)`を実装することを推奨します。理由:

-   `resolveAccount(...)`はランタイムパスです。資格情報が完全に具体化されていると仮定することが許可され、必要なシークレットが欠落している場合は迅速に失敗できます。
-   `openclaw status`, `openclaw status --all`, `openclaw channels status`, `openclaw channels resolve`, およびdoctor/設定修復フローなどの読み取り専用コマンドパスは、設定を記述するためだけにランタイム資格情報を具体化する必要はありません。

推奨される`inspectAccount(...)`の動作:

-   記述的なアカウント状態のみを返します。
-   `enabled`と`configured`を保持します。
-   関連する場合、以下のような資格情報ソース/ステータスフィールドを含めます:
    -   `tokenSource`, `tokenStatus`
    -   `botTokenSource`, `botTokenStatus`
    -   `appTokenSource`, `appTokenStatus`
    -   `signingSecretSource`, `signingSecretStatus`
-   読み取り専可用の可用性を報告するためだけに生のトークン値を返す必要はありません。`tokenStatus: "available"`（および一致するソースフィールド）を返すだけで、ステータス形式のコマンドには十分です。
-   資格情報がSecretRef経由で設定されているが現在のコマンドパスで利用できない場合は、`configured_unavailable`を使用します。

これにより、読み取り専用コマンドはクラッシュしたりアカウントを未設定と誤報告したりする代わりに、「設定済みだがこのコマンドパスでは利用不可」と報告できます。パフォーマンスに関する注意:

-   プラグイン検出とマニフェストメタデータは、突発的な起動/再読み込み作業を減らすために短いプロセス内キャッシュを使用します。
-   `OPENCLAW_DISABLE_PLUGIN_DISCOVERY_CACHE=1`または`OPENCLAW_DISABLE_PLUGIN_MANIFEST_CACHE=1`を設定してこれらのキャッシュを無効にできます。
-   キャッシュウィンドウは`OPENCLAW_PLUGIN_DISCOVERY_CACHE_MS`と`OPENCLAW_PLUGIN_MANIFEST_CACHE_MS`で調整できます。

## 検出と優先順位

OpenClawは以下の順序でスキャンします:

1.  設定パス

-   `plugins.load.paths` (ファイルまたはディレクトリ)

2.  ワークスペース拡張機能

-   `/.openclaw/extensions/*.ts`
-   `/.openclaw/extensions/*/index.ts`

3.  グローバル拡張機能

-   `~/.openclaw/extensions/*.ts`
-   `~/.openclaw/extensions/*/index.ts`

4.  バンドル拡張機能 (OpenClawに同梱、ほとんどはデフォルト無効)

-   `/extensions/*`

ほとんどのバンドルプラグインは、`plugins.entries..enabled`または`openclaw plugins enable `で明示的に有効にする必要があります。デフォルトで有効なバンドルプラグインの例外:

-   `device-pair`
-   `phone-control`
-   `talk-voice`
-   アクティブなメモリスロットプラグイン (デフォルトスロット: `memory-core`)

インストールされたプラグインはデフォルトで有効ですが、同じ方法で無効にできます。セキュリティ強化に関する注意:

-   `plugins.allow`が空で、非バンドルプラグインが検出可能な場合、OpenClawはプラグインIDとソースを含む起動警告をログに記録します。
-   候補パスは、検出許可前にセキュリティチェックされます。以下の場合、OpenClawは候補をブロックします:
    -   拡張機能エントリがプラグインルート外に解決される場合（シンボリックリンク/パストラバーサルエスケープを含む）、
    -   プラグインルート/ソースパスがワールド書き込み可能である場合、
    -   非バンドルプラグインのパス所有権が疑わしい場合（POSIX所有者が現在のuidでもrootでもない）。
-   インストール/ロードパスの出所がないロード済み非バンドルプラグインは警告を発し、信頼を固定（`plugins.allow`）またはインストール追跡（`plugins.installs`）できるようにします。

各プラグインは、そのルートに`openclaw.plugin.json`ファイルを含める必要があります。パスがファイルを指す場合、プラグインルートはファイルのディレクトリであり、マニフェストを含んでいる必要があります。複数のプラグインが同じIDに解決する場合、上記の順序で最初に一致したものが優先され、優先順位の低いコピーは無視されます。

### パッケージパック

プラグインディレクトリには、`openclaw.extensions`を含む`package.json`が含まれる場合があります:

```json
{
  "name": "my-pack",
  "openclaw": {
    "extensions": ["./src/safety.ts", "./src/tools.ts"]
  }
}
```

各エントリはプラグインになります。パックが複数の拡張機能をリストする場合、プラグインIDは`name/`になります。プラグインがnpm依存関係をインポートする場合は、そのディレクトリに`node_modules`が利用可能になるようにインストールしてください（`npm install` / `pnpm install`）。セキュリティガードレール: すべての`openclaw.extensions`エントリは、シンボリックリンク解決後もプラグインディレクトリ内に留まる必要があります。パッケージディレクトリ外にエスケープするエントリは拒否されます。セキュリティに関する注意: `openclaw plugins install`は`npm install --ignore-scripts`（ライフサイクルスクリプトなし）でプラグイン依存関係をインストールします。プラグイン依存関係ツリーを「純粋なJS/TS」に保ち、`postinstall`ビルドを必要とするパッケージを避けてください。

### チャネルカタログメタデータ

チャネルプラグインは、`openclaw.channel`経由でオンボーディングメタデータを、`openclaw.install`経由でインストールヒントを広告できます。これにより、コアカタログはデータフリーのまま維持されます。例:

```json
{
  "name": "@openclaw/nextcloud-talk",
  "openclaw": {
    "extensions": ["./index.ts"],
    "channel": {
      "id": "nextcloud-talk",
      "label": "Nextcloud Talk",
      "selectionLabel": "Nextcloud Talk (self-hosted)",
      "docsPath": "/channels/nextcloud-talk",
      "docsLabel": "nextcloud-talk",
      "blurb": "Nextcloud Talkウェブフックボットによるセルフホストチャット。",
      "order": 65,
      "aliases": ["nc-talk", "nc"]
    },
    "install": {
      "npmSpec": "@openclaw/nextcloud-talk",
      "localPath": "extensions/nextcloud-talk",
      "defaultChoice": "npm"
    }
  }
}
```

OpenClawは**外部チャネルカタログ**（例: MPMレジストリエクスポート）もマージできます。以下のいずれかにJSONファイルを配置します:

-   `~/.openclaw/mpm/plugins.json`
-   `~/.openclaw/mpm/catalog.json`
-   `~/.openclaw/plugins/catalog.json`

または、`OPENCLAW_PLUGIN_CATALOG_PATHS`（または`OPENCLAW_MPM_CATALOG_PATHS`）を1つ以上のJSONファイル（カンマ/セミコロン/`PATH`区切り）にポイントします。各ファイルには`{ "entries": [ { "name": "@scope/pkg", "openclaw": { "channel": {...}, "install": {...} } } ] }`が含まれている必要があります。

## プラグインID

デフォルトのプラグインID:

-   パッケージパック: `package.json`の`name`
-   スタンドアロンファイル: ファイルベース名（`~/.../voice-call.ts` → `voice-call`）

プラグインが`id`をエクスポートする場合、OpenClawはそれを使用しますが、設定されたIDと一致しない場合は警告します。

## 設定

```json
{
  plugins: {
    enabled: true,
    allow: ["voice-call"],
    deny: ["untrusted-plugin"],
    load: { paths: ["~/Projects/oss/voice-call-extension"] },
    entries: {
      "voice-call": { enabled: true, config: { provider: "twilio" } },
    },
  },
}
```

フィールド:

-   `enabled`: マスタートグル（デフォルト: true）
-   `allow`: 許可リスト（オプション）
-   `deny`: 拒否リスト（オプション；拒否が優先）
-   `load.paths`: 追加のプラグインファイル/ディレクトリ
-   `slots`: `memory`や`contextEngine`などの排他的スロットセレクター
-   `entries.`: プラグインごとのトグル + 設定

設定変更には**gatewayの再起動が必要**です。検証ルール（厳格）:

-   `entries`、`allow`、`deny`、`slots`内の未知のプラグインIDは**エラー**です。
-   プラグインマニフェストがチャネルIDを宣言していない限り、未知の`channels.`キーは**エラー**です。
-   プラグイン設定は、`openclaw.plugin.json`（`configSchema`）に埋め込まれたJSON Schemaを使用して検証されます。
-   プラグインが無効な場合、その設定は保持され、**警告**が出力されます。

## プラグインスロット (排他的カテゴリ)

一部のプラグインカテゴリは**排他的**です（一度に1つだけアクティブ）。`plugins.slots`を使用して、どのプラグインがスロットを所有するかを選択します:

```json
{
  plugins: {
    slots: {
      memory: "memory-core", // または "none" でメモリプラグインを無効化
      contextEngine: "legacy", // または "lossless-claw" などのプラグインID
    },
  },
}
```

サポートされている排他スロット:

-   `memory`: アクティブなメモリプラグイン（`"none"`はメモリプラグインを無効化）
-   `contextEngine`: アクティブなコンテキストエンジンプラグイン（`"legacy"`は組み込みデフォルト）

複数のプラグインが`kind: "memory"`または`kind: "context-engine"`を宣言する場合、そのスロットに対して選択されたプラグインのみが読み込まれます。他は診断情報付きで無効化されます。

### コンテキストエンジンプラグイン

コンテキストエンジンプラグインは、取り込み、組み立て、圧縮のためのセッションコンテキストオーケストレーションを所有します。プラグインから`api.registerContextEngine(id, factory)`で登録し、`plugins.slots.contextEngine`でアクティブなエンジンを選択します。これは、プラグインがメモリ検索やフックを追加するだけでなく、デフォルトのコンテキストパイプラインを置き換えたり拡張したりする必要がある場合に使用します。

## コントロールUI (スキーマ + ラベル)

コントロールUIは、`config.schema`（JSON Schema + `uiHints`）を使用してより良いフォームをレンダリングします。OpenClawは、検出されたプラグインに基づいてランタイムで`uiHints`を拡張します:

-   `plugins.entries.` / `.enabled` / `.config`に対してプラグインごとのラベルを追加
-   オプションのプラグイン提供設定フィールドヒントを以下にマージ: `plugins.entries..config.`

プラグイン設定フィールドに適切なラベル/プレースホルダーを表示させたい（およびシークレットを機密としてマークしたい）場合は、プラグインマニフェストのJSON Schemaと共に`uiHints`を提供してください。例:

```json
{
  "id": "my-plugin",
  "configSchema": {
    "type": "object",
    "additionalProperties": false,
    "properties": {
      "apiKey": { "type": "string" },
      "region": { "type": "string" }
    }
  },
  "uiHints": {
    "apiKey": { "label": "APIキー", "sensitive": true },
    "region": { "label": "リージョン", "placeholder": "us-east-1" }
  }
}
```

## CLI

```bash
openclaw plugins list
openclaw plugins info <id>
openclaw plugins install <path>                 # ローカルファイル/ディレクトリを ~/.openclaw/extensions/<id> にコピー
openclaw plugins install ./extensions/voice-call # 相対パス可
openclaw plugins install ./plugin.tgz           # ローカルtarballからインストール
openclaw plugins install ./plugin.zip           # ローカルzipからインストール
openclaw plugins install -l ./extensions/voice-call # 開発用にリンク（コピーなし）
openclaw plugins install @openclaw/voice-call # npmからインストール
openclaw plugins install @openclaw/voice-call --pin # 正確な解決済み name@version を保存
openclaw plugins update <id>
openclaw plugins update --all
openclaw plugins enable <id>
openclaw plugins disable <id>
openclaw plugins doctor
```

`plugins update`は、`plugins.installs`で追跡されているnpmインストールに対してのみ機能します。保存された整合性メタデータが更新間で変更された場合、OpenClawは警告を発し確認を求めます（グローバル`--yes`を使用してプロンプトをバイパス）。プラグインは独自のトップレベルコマンドも登録できます（例: `openclaw voicecall`）。

## プラグインAPI (概要)

プラグインは以下をエクスポートします:

-   関数: `(api) => { ... }`
-   オブジェクト: `{ id, name, configSchema, register(api) { ... } }`

コンテキストエンジンプラグインは、ランタイム所有のコンテキストマネージャーも登録できます:

```bash
export default function (api) {
  api.registerContextEngine("lossless-claw", () => ({
    info: { id: "lossless-claw", name: "Lossless Claw", ownsCompaction: true },
    async ingest() {
      return { ingested: true };
    },
    async assemble({ messages }) {
      return { messages, estimatedTokens: 0 };
    },
    async compact() {
      return { ok: true, compacted: false };
    },
  }));
}
```

その後、設定で有効にします:

```json
{
  plugins: {
    slots: {
      contextEngine: "lossless-claw",
    },
  },
}
```

## プラグインフック

プラグインはランタイムにフックを登録できます。これにより、プラグインは別個のフックパックインストールなしでイベント駆動型自動化をバンドルできます。

### 例

```bash
export default function register(api) {
  api.registerHook(
    "command:new",
    async () => {
      // フックロジックをここに記述。
    },
    {
      name: "my-plugin.command-new",
      description: "/newが呼び出されたときに実行",
    },
  );
}
```

注意点:

-   フックは`api.registerHook(...)`経由で明示的に登録してください。
-   フック適格性ルールは引き続き適用されます（OS/バイナリ/環境/設定要件）。
-   プラグイン管理フックは`openclaw hooks list`に`plugin:`として表示されます。
-   `openclaw hooks`経由でプラグイン管理フックを有効/無効にすることはできません。代わりにプラグイン自体を有効/無効にしてください。

### エージェントライフサイクルフック (api.on)

型付きランタイムライフサイクルフックには、`api.on(...)`を使用します:

```bash
export default function register(api) {
  api.on(
    "before_prompt_build",
    (event, ctx) => {
      return {
        prependSystemContext: "会社のスタイルガイドに従ってください。",
      };
    },
    { priority: 10 },
  );
}
```

プロンプト構築の重要なフック:

-   `before_model_resolve`: セッション読み込み前に実行（`messages`は利用不可）。これを使用して`modelOverride`または`providerOverride`を確定的にオーバーライドします。
-   `before_prompt_build`: セッション読み込み後に実行（`messages`が利用可能）。これを使用してプロンプト入力を形成します。
-   `before_agent_start`: レガシー互換性フック。上記の2つの明示的なフックを優先してください。

コア強制フックポリシー:

-   オペレーターは、`plugins.entries..hooks.allowPromptInjection: false`でプラグインごとにプロンプト変更フックを無効にできます。
-   無効にすると、OpenClawは`before_prompt_build`をブロックし、レガシー`before_agent_start`から返されたプロンプト変更フィールドを無視しながら、レガシー`modelOverride`と`providerOverride`を保持します。

`before_prompt_build`結果フィールド:

-   `prependContext`: この実行のユーザープロンプトの前にテキストを追加します。ターン固有または動的コンテンツに最適です。
-   `systemPrompt`: 完全なシステムプロンプトのオーバーライド。
-   `prependSystemContext`: 現在のシステムプロンプトの前にテキストを追加します。
-   `appendSystemContext`: 現在のシステムプロンプトの後にテキストを追加します。

組み込みランタイムでのプロンプト構築順序:

1.  `prependContext`をユーザープロンプトに適用。
2.  提供された場合、`systemPrompt`オーバーライドを適用。
3.  `prependSystemContext + 現在のシステムプロンプト + appendSystemContext`を適用。

マージと優先順位に関する注意:

-   フックハンドラーは優先度順（高いものから）に実行されます。
-   マージされたコンテキストフィールドの場合、値は実行順に連結されます。
-   `before_prompt_build`の値は、レガシー`before_agent_start`フォールバック値の前に適用されます。

移行ガイダンス:

-   静的ガイダンスを`prependContext`から`prependSystemContext`（または`appendSystemContext`）に移動して、プロバイダーが安定したシステムプレフィックスコンテンツをキャッシュできるようにします。
-   ユーザーメッセージに結び付けるべきターンごとの動的コンテンツには`prependContext`を保持します。

## プロバイダープラグイン (モデル認証)

プラグインは**モデルプロバイダー認証**フローを登録でき、ユーザーはOpenClaw内でOAuthやAPIキー設定を実行できます（外部スクリプト不要）。`api.registerProvider(...)`でプロバイダーを登録します。各プロバイダーは1つ以上の認証方法（OAuth、APIキー、デバイスコードなど）を公開します。これらの方法は以下を実現します:

-   `openclaw models auth login --provider  [--method ]`

例:

```
api.registerProvider({
  id: "acme",
  label: "AcmeAI",
  auth: [
    {
      id: "oauth",
      label: "OAuth",
      kind: "oauth",
      run: async (ctx) => {
        // OAuthフローを実行し、認証プロファイルを返します。
        return {
          profiles: [
            {
              profileId: "acme:default",
              credential: {
                type: "oauth",
                provider: "acme",
                access: "...",
                refresh: "...",
                expires: Date.now() + 3600 * 1000,
              },
            },
          ],
          defaultModel: "acme/opus-1",
        };
      },
    },
  ],
});
```

注意点:

-   `run`は`prompter`、`runtime`、`openUrl`、`oauth.createVpsAwareHandlers`ヘルパーを含む`ProviderAuthContext`を受け取ります。
-   デフォルトモデルやプロバイダー設定を追加する必要がある場合は`configPatch`を返します。
-   `--set-default`がエージェントデフォルトを更新できるように`defaultModel`を返します。

### メッセージングチャネルの登録

プラグインは、組み込みチャネル（WhatsApp、Telegramなど）のように動作する**チャネルプラグイン**を登録できます。チャネル設定は`channels.`下に存在し、チャネルプラグインコードによって検証されます。

```typescript
const myChannel = {
  id: "acmechat",
  meta: {
    id: "acmechat",
    label: "AcmeChat",
    selectionLabel: "AcmeChat (API)",
    docs