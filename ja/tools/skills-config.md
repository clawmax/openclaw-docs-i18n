

  スキル

  
# スキル設定

すべてのスキル関連の設定は、`~/.openclaw/openclaw.json` 内の `skills` の下に記述します。

```json
{
  skills: {
    allowBundled: ["gemini", "peekaboo"],
    load: {
      extraDirs: ["~/Projects/agent-scripts/skills", "~/Projects/oss/some-skill-pack/skills"],
      watch: true,
      watchDebounceMs: 250,
    },
    install: {
      preferBrew: true,
      nodeManager: "npm", // npm | pnpm | yarn | bun (Gateway runtime still Node; bun not recommended)
    },
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // or plaintext string
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

## フィールド

-   `allowBundled`: **バンドル済み**スキルのみに対するオプションの許可リスト。設定すると、リスト内のバンドル済みスキルのみが有効になります（管理対象/ワークスペーススキルは影響を受けません）。
-   `load.extraDirs`: スキャンする追加のスキルディレクトリ（優先度は最も低い）。
-   `load.watch`: スキルフォルダを監視し、スキルのスナップショットを更新します（デフォルト: true）。
-   `load.watchDebounceMs`: スキル監視イベントのデバウンス時間（ミリ秒）（デフォルト: 250）。
-   `install.preferBrew`: 利用可能な場合はbrewインストーラーを優先します（デフォルト: true）。
-   `install.nodeManager`: Nodeインストーラーの優先設定 (`npm` | `pnpm` | `yarn` | `bun`, デフォルト: npm）。これは**スキルのインストール**のみに影響します。Gatewayランタイムは引き続きNodeであるべきです（WhatsApp/TelegramではBunは推奨されません）。
-   `entries.`: スキルごとのオーバーライド設定。

スキルごとのフィールド:

-   `enabled`: `false` に設定すると、スキルがバンドル済み/インストール済みであっても無効化します。
-   `env`: エージェント実行時に注入される環境変数（既に設定されていない場合のみ）。
-   `apiKey`: 主要な環境変数を宣言するスキル用のオプションの便利機能。プレーンテキスト文字列またはSecretRefオブジェクト (`{ source, provider, id }`) をサポートします。

## 注意事項

-   `entries` の下のキーは、デフォルトでスキル名にマッピングされます。スキルが `metadata.openclaw.skillKey` を定義している場合は、そのキーを代わりに使用してください。
-   スキルへの変更は、監視が有効な場合、次のエージェントターンで検知されます。

### サンドボックス化されたスキル + 環境変数

セッションが**サンドボックス化**されている場合、スキルプロセスはDocker内で実行されます。サンドボックスはホストの `process.env` を**継承しません**。以下のいずれかを使用してください:

-   `agents.defaults.sandbox.docker.env` (またはエージェントごとの `agents.list[].sandbox.docker.env`)
-   カスタムサンドボックスイメージに環境変数を組み込む

グローバルな `env` および `skills.entries..env/apiKey` は、**ホスト**での実行にのみ適用されます。

[スキル](./skills.md)[ClawHub](./clawhub.md)

---