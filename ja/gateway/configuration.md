

  設定と運用

  
# 設定

OpenClaw は、オプションの **JSON5** 設定を `~/.openclaw/openclaw.json` から読み込みます。ファイルが存在しない場合、OpenClaw は安全なデフォルト値を使用します。設定を追加する一般的な理由は以下の通りです:

-   チャネルを接続し、誰がボットにメッセージを送信できるかを制御する
-   モデル、ツール、サンドボックス化、または自動化 (cron、フック) を設定する
-   セッション、メディア、ネットワーキング、または UI を調整する

利用可能なすべてのフィールドについては、[完全なリファレンス](./configuration-reference.md) を参照してください。

> **💡** **設定が初めてですか？** インタラクティブなセットアップには `openclaw onboard` から始めるか、完全なコピー＆ペースト設定については [設定例](./configuration-examples.md) ガイドを確認してください。

## 最小限の設定

```
// ~/.openclaw/openclaw.json
{
  agents: { defaults: { workspace: "~/.openclaw/workspace" } },
  channels: { whatsapp: { allowFrom: ["+15555550123"] } },
}
```

## 設定の編集

```bash
openclaw onboard       # 完全セットアップウィザード
openclaw configure     # 設定ウィザード
```

```bash
openclaw config get agents.defaults.workspace
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config unset tools.web.search.apiKey
```

[http://127.0.0.1:18789](http://127.0.0.1:18789) を開き、**Config** タブを使用します。コントロールUIは設定スキーマからフォームをレンダリングし、**Raw JSON** エディタをエスケープハッチとして提供します。

`~/.openclaw/openclaw.json` を直接編集します。ゲートウェイはファイルを監視し、変更を自動的に適用します ([ホットリロード](#config-hot-reload) を参照)。

## 厳密な検証

> **⚠️** OpenClaw は、スキーマに完全に一致する設定のみを受け入れます。未知のキー、不正な形式の型、または無効な値があると、ゲートウェイは **起動を拒否します**。唯一のルートレベルの例外は `$schema` (文字列) であり、エディタが JSON スキーマメタデータを添付できるようにします。

 検証が失敗した場合:

-   ゲートウェイは起動しません
-   診断コマンドのみが機能します (`openclaw doctor`, `openclaw logs`, `openclaw health`, `openclaw status`)
-   正確な問題を確認するには `openclaw doctor` を実行します
-   修復を適用するには `openclaw doctor --fix` (または `--yes`) を実行します

## 一般的なタスク

各チャネルには、`channels.` の下に独自の設定セクションがあります。セットアップ手順については、専用のチャネルページを参照してください:

-   [WhatsApp](../channels/whatsapp.md) — `channels.whatsapp`
-   [Telegram](../channels/telegram.md) — `channels.telegram`
-   [Discord](../channels/discord.md) — `channels.discord`
-   [Slack](../channels/slack.md) — `channels.slack`
-   [Signal](../channels/signal.md) — `channels.signal`
-   [iMessage](../channels/imessage.md) — `channels.imessage`
-   [Google Chat](../channels/googlechat.md) — `channels.googlechat`
-   [Mattermost](../channels/mattermost.md) — `channels.mattermost`
-   [MS Teams](../channels/msteams.md) — `channels.msteams`

すべてのチャネルは同じ DM ポリシーパターンを共有します:

```json
{
  channels: {
    telegram: {
      enabled: true,
      botToken: "123:abc",
      dmPolicy: "pairing",   // pairing | allowlist | open | disabled
      allowFrom: ["tg:123"], // only for allowlist/open
    },
  },
}
```

プライマリモデルとオプションのフォールバックを設定します:

```json
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-sonnet-4-5",
        fallbacks: ["openai/gpt-5.2"],
      },
      models: {
        "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
        "openai/gpt-5.2": { alias: "GPT" },
      },
    },
  },
}
```

-   `agents.defaults.models` はモデルカタログを定義し、`/model` の許可リストとして機能します。
-   モデル参照は `provider/model` 形式を使用します (例: `anthropic/claude-opus-4-6`)。
-   `agents.defaults.imageMaxDimensionPx` は、トランスクリプト/ツール画像のダウンスケーリングを制御します (デフォルト `1200`)。低い値は通常、スクリーンショットが多い実行時のビジョントークン使用量を削減します。
-   チャットでのモデル切り替えについては [Models CLI](../concepts/models.md) を、認証ローテーションとフォールバック動作については [Model Failover](../concepts/model-failover.md) を参照してください。
-   カスタム/セルフホストプロバイダについては、リファレンスの [カスタムプロバイダ](./configuration-reference.md#custom-providers-and-base-urls) を参照してください。

DM アクセスは、チャネルごとに `dmPolicy` で制御されます:

-   `"pairing"` (デフォルト): 未知の送信者は承認用のワンタイムペアリングコードを受け取ります
-   `"allowlist"`: `allowFrom` (またはペアリング済みの許可ストア) 内の送信者のみ
-   `"open"`: すべての受信 DM を許可します (`allowFrom: ["*"]` が必要)
-   `"disabled"`: すべての DM を無視します

グループの場合は、`groupPolicy` + `groupAllowFrom` またはチャネル固有の許可リストを使用します。チャネルごとの詳細については [完全なリファレンス](./configuration-reference.md#dm-and-group-access) を参照してください。

グループメッセージはデフォルトで **メンションを必要とします**。エージェントごとにパターンを設定します:

```json
{
  agents: {
    list: [
      {
        id: "main",
        groupChat: {
          mentionPatterns: ["@openclaw", "openclaw"],
        },
      },
    ],
  },
  channels: {
    whatsapp: {
      groups: { "*": { requireMention: true } },
    },
  },
}
```

-   **メタデータメンション**: ネイティブの @-メンション (WhatsApp のタップしてメンション、Telegram の @bot など)
-   **テキストパターン**: `mentionPatterns` 内の正規表現パターン
-   チャネルごとのオーバーライドとセルフチャットモードについては [完全なリファレンス](./configuration-reference.md#group-chat-mention-gating) を参照してください。

セッションは会話の継続性と分離を制御します:

```json
{
  session: {
    dmScope: "per-channel-peer",  // マルチユーザーに推奨
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
    reset: {
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
  },
}
```

-   `dmScope`: `main` (共有) | `per-peer` | `per-channel-peer` | `per-account-channel-peer`
-   `threadBindings`: スレッドバインドされたセッションルーティングのグローバルデフォルト (Discord は `/focus`, `/unfocus`, `/agents`, `/session idle`, `/session max-age` をサポート)。
-   スコープ、アイデンティティリンク、送信ポリシーについては [セッション管理](../concepts/session.md) を参照してください。
-   すべてのフィールドについては [完全なリファレンス](./configuration-reference.md#session) を参照してください。

エージェントセッションを分離された Docker コンテナで実行します:

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",  // off | non-main | all
        scope: "agent",    // session | agent | shared
      },
    },
  },
}
```

最初にイメージをビルドします: `scripts/sandbox-setup.sh`完全なガイドについては [サンドボックス化](./sandboxing.md) を、すべてのオプションについては [完全なリファレンス](./configuration-reference.md#sandbox) を参照してください。

```json
{
  agents: {
    defaults: {
      heartbeat: {
        every: "30m",
        target: "last",
      },
    },
  },
}
```

-   `every`: 期間文字列 (`30m`, `2h`)。無効にするには `0m` を設定します。
-   `target`: `last` | `whatsapp` | `telegram` | `discord` | `none`
-   `directPolicy`: DM スタイルのハートビートターゲットに対して `allow` (デフォルト) または `block`
-   完全なガイドについては [ハートビート](./heartbeat.md) を参照してください。

```json
{
  cron: {
    enabled: true,
    maxConcurrentRuns: 2,
    sessionRetention: "24h",
    runLog: {
      maxBytes: "2mb",
      keepLines: 2000,
    },
  },
}
```

-   `sessionRetention`: 完了した分離実行セッションを `sessions.json` から削除します (デフォルト `24h`; 無効にするには `false` を設定)。
-   `runLog`: `cron/runs/.jsonl` をサイズと保持行数で削除します。
-   機能の概要と CLI の例については [Cron ジョブ](../automation/cron-jobs.md) を参照してください。

ゲートウェイで HTTP ウェブフックエンドポイントを有効にします:

```json
{
  hooks: {
    enabled: true,
    token: "shared-secret",
    path: "/hooks",
    defaultSessionKey: "hook:ingress",
    allowRequestSessionKey: false,
    allowedSessionKeyPrefixes: ["hook:"],
    mappings: [
      {
        match: { path: "gmail" },
        action: "agent",
        agentId: "main",
        deliver: true,
      },
    ],
  },
}
```

セキュリティ上の注意:

-   すべてのフック/ウェブフックペイロードの内容を信頼できない入力として扱います。
-   厳密にスコープされたデバッグを行わない限り、安全でないコンテンツのバイパスフラグは無効にしておきます (`hooks.gmail.allowUnsafeExternalContent`, `hooks.mappings[].allowUnsafeExternalContent`)。
-   フック駆動のエージェントの場合は、強力な最新のモデル階層と厳格なツールポリシーを優先します (例えば、メッセージングのみに加え、可能な場合はサンドボックス化)。

すべてのマッピングオプションと Gmail 統合については [完全なリファレンス](./configuration-reference.md#hooks) を参照してください。

複数の分離されたエージェントを別々のワークスペースとセッションで実行します:

```json
{
  agents: {
    list: [
      { id: "home", default: true, workspace: "~/.openclaw/workspace-home" },
      { id: "work", workspace: "~/.openclaw/workspace-work" },
    ],
  },
  bindings: [
    { agentId: "home", match: { channel: "whatsapp", accountId: "personal" } },
    { agentId: "work", match: { channel: "whatsapp", accountId: "biz" } },
  ],
}
```

バインディングルールとエージェントごとのアクセスプロファイルについては [マルチエージェント](../concepts/multi-agent.md) と [完全なリファレンス](./configuration-reference.md#multi-agent-routing) を参照してください。

`$include` を使用して大規模な設定を整理します:

```
// ~/.openclaw/openclaw.json
{
  gateway: { port: 18789 },
  agents: { $include: "./agents.json5" },
  broadcast: {
    $include: ["./clients/a.json5", "./clients/b.json5"],
  },
}
```

-   **単一ファイル**: 包含オブジェクトを置き換えます
-   **ファイルの配列**: 順番にディープマージされます (後勝ち)
-   **兄弟キー**: インクルード後にマージされます (インクルードされた値を上書き)
-   **ネストされたインクルード**: 最大10レベルまでサポート
-   **相対パス**: インクルード元のファイルからの相対パスで解決されます
-   **エラーハンドリング**: 欠落ファイル、パースエラー、循環インクルードに対して明確なエラーを表示

## 設定ホットリロード

ゲートウェイは `~/.openclaw/openclaw.json` を監視し、変更を自動的に適用します — ほとんどの設定で手動での再起動は不要です。

### リロードモード

| モード | 動作 |
| --- | --- |
| **`hybrid`** (デフォルト) | 安全な変更を即座にホット適用します。重要な変更には自動的に再起動します。 |
| **`hot`** | 安全な変更のみをホット適用します。再起動が必要な場合は警告をログに記録します — ユーザーが対応します。 |
| **`restart`** | 設定変更のたびに、安全かどうかに関わらずゲートウェイを再起動します。 |
| **`off`** | ファイル監視を無効にします。変更は次回の手動再起動で有効になります。 |

```json
{
  gateway: {
    reload: { mode: "hybrid", debounceMs: 300 },
  },
}
```

### ホット適用されるもの vs 再起動が必要なもの

ほとんどのフィールドはダウンタイムなしでホット適用されます。`hybrid` モードでは、再起動が必要な変更は自動的に処理されます。

| カテゴリ | フィールド | 再起動が必要？ |
| --- | --- | --- |
| チャネル | `channels.*`, `web` (WhatsApp) — すべての組み込みおよび拡張チャネル | いいえ |
| エージェント & モデル | `agent`, `agents`, `models`, `routing` | いいえ |
| 自動化 | `hooks`, `cron`, `agent.heartbeat` | いいえ |
| セッション & メッセージ | `session`, `messages` | いいえ |
| ツール & メディア | `tools`, `browser`, `skills`, `audio`, `talk` | いいえ |
| UI & その他 | `ui`, `logging`, `identity`, `bindings` | いいえ |
| ゲートウェイサーバー | `gateway.*` (ポート、バインド、認証、tailscale、TLS、HTTP) | **はい** |
| インフラストラクチャ | `discovery`, `canvasHost`, `plugins` | **はい** |

> **ℹ️** `gateway.reload` と `gateway.remote` は例外です — これらを変更しても **再起動はトリガーされません**。

## 設定 RPC (プログラムによる更新)

> **ℹ️** コントロールプレーンの書き込み RPC (`config.apply`, `config.patch`, `update.run`) は、`deviceId+clientIp` ごとに **60秒あたり3リクエスト** にレート制限されます。制限されると、RPC は `UNAVAILABLE` と `retryAfterMs` を返します。

 

完全な設定を検証 + 書き込み、ゲートウェイを再起動することを1ステップで行います。

`config.apply` は **設定全体を置き換えます**。部分的な更新には `config.patch` を、単一キーの更新には `openclaw config set` を使用します。

パラメータ:

-   `raw` (string) — 設定全体の JSON5 ペイロード
-   `baseHash` (オプション) — `config.get` からの設定ハッシュ (設定が存在する場合は必須)
-   `sessionKey` (オプション) — 再起動後のウェイクアップ ping 用のセッションキー
-   `note` (オプション) — 再起動センチネル用のメモ
-   `restartDelayMs` (オプション) — 再起動前の遅延 (デフォルト 2000)

再起動リクエストは、すでに保留中/実行中の間に統合され、再起動サイクル間には30秒のクールダウンが適用されます。

```bash
openclaw gateway call config.get --params '{}'  # payload.hash を取得
openclaw gateway call config.apply --params '{
  "raw": "{ agents: { defaults: { workspace: \"~/.openclaw/workspace\" } } }",
  "baseHash": "<hash>",
  "sessionKey": "agent:main:whatsapp:dm:+15555550123"
}'
```

既存の設定に部分的な更新をマージします (JSON マージパッチのセマンティクス):

-   オブジェクトは再帰的にマージされます
-   `null` はキーを削除します
-   配列は置き換えられます

パラメータ:

-   `raw` (string) — 変更するキーのみを含む JSON5
-   `baseHash` (必須) — `config.get` からの設定ハッシュ
-   `sessionKey`, `note`, `restartDelayMs` — `config.apply` と同じ

再起動動作は `config.apply` と一致します: 保留中の再起動を統合し、再起動サイクル間には30秒のクールダウンが適用されます。

```bash
openclaw gateway call config.patch --params '{
  "raw": "{ channels: { telegram: { groups: { \"*\": { requireMention: false } } } } }",
  "baseHash": "<hash>"
}'
```

## 環境変数

OpenClaw は親プロセスから環境変数を読み取ります。さらに以下からも読み取ります:

-   現在の作業ディレクトリの `.env` (存在する場合)
-   `~/.openclaw/.env` (グローバルフォールバック)

どちらのファイルも既存の環境変数を上書きしません。設定内でインライン環境変数を設定することもできます:

```json
{
  env: {
    OPENROUTER_API_KEY: "sk-or-...",
    vars: { GROQ_API_KEY: "gsk-..." },
  },
}
```

有効にされていて、必要なキーが設定されていない場合、OpenClaw はログインシェルを実行し、欠落しているキーのみをインポートします:

```json
{
  env: {
    shellEnv: { enabled: true, timeoutMs: 15000 },
  },
}
```

環境変数相当: `OPENCLAW_LOAD_SHELL_ENV=1`

 

任意の設定文字列値で環境変数を `${VAR_NAME}` で参照できます:

```json
{
  gateway: { auth: { token: "${OPENCLAW_GATEWAY_TOKEN}" } },
  models: { providers: { custom: { apiKey: "${CUSTOM_API_KEY}" } } },
}
```

ルール:

-   大文字の名前のみ一致: `[A-Z_][A-Z0-9_]*`
-   欠落/空の変数はロード時にエラーをスローします
-   リテラル出力には `$${VAR}` でエスケープします
-   `$include` ファイル内でも動作します
-   インライン置換: `"${BASE}/v1"` → `"https://api.example.com/v1"`

 

SecretRef オブジェクトをサポートするフィールドでは、以下を使用できます:

```json
{
  models: {
    providers: {
      openai: { apiKey: { source: "env", provider: "default", id: "OPENAI_API_KEY" } },
    },
  },
  skills: {
    entries: {
      "nano-banana-pro": {
        apiKey: {
          source: "file",
          provider: "filemain",
          id: "/skills/entries/nano-banana-pro/apiKey",
        },
      },
    },
  },
  channels: {
    googlechat: {
      serviceAccountRef: {
        source: "exec",
        provider: "vault",
        id: "channels/googlechat/serviceAccount",
      },
    },
  },
}
```

SecretRef の詳細 (`env`/`file`/`exec` 用の `secrets.providers` を含む) は [シークレット管理](./secrets.md) にあります。サポートされている認証情報パスは [SecretRef 認証情報サーフェス](../reference/secretref-credential-surface.md) にリストされています。

 完全な優先順位とソースについては [環境](../help/environment.md) を参照してください。

## 完全なリファレンス

フィールドごとの完全なリファレンスについては、**[設定リファレンス](./configuration-reference.md)** を参照してください。

* * *

*関連: [設定例](./configuration-examples.md) · [設定リファレンス](./configuration-reference.md) · [Doctor](./doctor.md)*

[ゲートウェイ ランブック](../gateway.md)[設定リファレンス](./configuration-reference.md)