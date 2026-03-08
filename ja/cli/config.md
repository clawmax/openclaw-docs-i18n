

  CLI コマンド

  
# config

設定ヘルパー: パスによる値の取得/設定/解除/検証を行い、アクティブな設定ファイルを表示します。サブコマンドなしで実行すると設定ウィザードが開きます (`openclaw configure` と同じ)。

## 例

```bash
openclaw config file
openclaw config get browser.executablePath
openclaw config set browser.executablePath "/usr/bin/google-chrome"
openclaw config set agents.defaults.heartbeat.every "2h"
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
openclaw config unset tools.web.search.apiKey
openclaw config validate
openclaw config validate --json
```

## パス

パスはドット表記またはブラケット表記を使用します:

```bash
openclaw config get agents.defaults.workspace
openclaw config get agents.list[0].id
```

特定のエージェントを対象にするには、エージェントリストのインデックスを使用します:

```bash
openclaw config get agents.list
openclaw config set agents.list[1].tools.exec.node "node-id-or-name"
```

## 値

値は可能な限り JSON5 としてパースされます。それ以外の場合は文字列として扱われます。JSON5 パースを必須にするには `--strict-json` を使用します。`--json` はレガシーエイリアスとして引き続きサポートされています。

```bash
openclaw config set agents.defaults.heartbeat.every "0m"
openclaw config set gateway.port 19001 --strict-json
openclaw config set channels.whatsapp.groups '["*"]' --strict-json
```

## サブコマンド

-   `config file`: アクティブな設定ファイルのパスを表示します (`OPENCLAW_CONFIG_PATH` またはデフォルトの場所から解決されます)。

編集後はゲートウェイを再起動してください。

## 検証

ゲートウェイを起動せずに、現在の設定をアクティブなスキーマに対して検証します。

```bash
openclaw config validate
openclaw config validate --json
```

[completion](./completion.md)[configure](./configure.md)

---