

  CLI コマンド

  
# models

モデルの検出、スキャン、設定（デフォルトモデル、フォールバック、認証プロファイル）。関連項目:

-   プロバイダーとモデル: [Models](../providers/models.md)
-   プロバイダー認証設定: [Getting started](../start/getting-started.md)

## よく使うコマンド

```bash
openclaw models status
openclaw models list
openclaw models set <model-or-alias>
openclaw models scan
```

`openclaw models status` は、解決されたデフォルト/フォールバックと認証の概要を表示します。プロバイダー使用状況のスナップショットが利用可能な場合、OAuth/トークン状態セクションにはプロバイダー使用状況ヘッダーが含まれます。`--probe` を追加すると、設定された各プロバイダープロファイルに対してライブ認証プローブを実行します。プローブは実際のリクエストです（トークンを消費し、レート制限をトリガーする可能性があります）。設定されたエージェントのモデル/認証状態を検査するには `--agent ` を使用します。省略した場合、このコマンドは `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` が設定されていればそれを使用し、それ以外の場合は設定されたデフォルトエージェントを使用します。注意点:

-   `models set <model-or-alias>` は `provider/model` またはエイリアスを受け付けます。
-   モデル参照は、**最初の** `/` で分割して解析されます。モデルIDに `/` が含まれる場合（OpenRouterスタイル）、プロバイダープレフィックスを含めてください（例: `openrouter/moonshotai/kimi-k2`）。
-   プロバイダーを省略した場合、OpenClawは入力をエイリアスまたは**デフォルトプロバイダー**のモデルとして扱います（モデルIDに `/` が含まれていない場合のみ機能します）。
-   `models status` は、認証出力で非シークレットのプレースホルダー（例: `OPENAI_API_KEY`, `secretref-managed`, `minimax-oauth`, `qwen-oauth`, `ollama-local`）に対して `marker()` を表示し、シークレットとしてマスクしない場合があります。

### models status

オプション:

-   `--json`
-   `--plain`
-   `--check` (終了コード 1=期限切れ/欠落, 2=期限切れ間近)
-   `--probe` (設定された認証プロファイルのライブプローブ)
-   `--probe-provider ` (1つのプロバイダーをプローブ)
-   `--probe-profile ` (プロファイルIDを繰り返しまたはカンマ区切りで指定)
-   `--probe-timeout `
-   `--probe-concurrency `
-   `--probe-max-tokens `
-   `--agent ` (設定されたエージェントID; `OPENCLAW_AGENT_DIR`/`PI_CODING_AGENT_DIR` を上書き)

## エイリアス + フォールバック

```bash
openclaw models aliases list
openclaw models fallbacks list
```

## 認証プロファイル

```bash
openclaw models auth add
openclaw models auth login --provider <id>
openclaw models auth setup-token
openclaw models auth paste-token
```

`models auth login` は、プロバイダープラグインの認証フロー（OAuth/APIキー）を実行します。インストールされているプロバイダーを確認するには `openclaw plugins list` を使用してください。注意点:

-   `setup-token` は、セットアップトークンの値の入力を促します（任意のマシンで `claude setup-token` を実行して生成します）。
-   `paste-token` は、他の場所や自動化から生成されたトークン文字列を受け付けます。
-   Anthropic ポリシーに関する注意: セットアップトークンのサポートは技術的な互換性のためです。Anthropicは過去にClaude Code外での一部のサブスクリプション使用をブロックしたことがあるため、広く使用する前に現在の利用規約を確認してください。

[message](./message.md)[node](./node.md)