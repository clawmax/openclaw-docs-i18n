

  モデルの概念

  
# Models CLI

認証プロファイルのローテーション、クールダウン、およびそれらがフォールバックとどのように相互作用するかについては、[/concepts/model-failover](./model-failover.md) を参照してください。プロバイダーの概要と例については、[/concepts/model-providers](./model-providers.md) を参照してください。

## モデル選択の仕組み

OpenClawは以下の順序でモデルを選択します：

1.  **プライマリ**モデル (`agents.defaults.model.primary` または `agents.defaults.model`)。
2.  `agents.defaults.model.fallbacks` 内の**フォールバック**（順序通り）。
3.  **プロバイダー認証フェイルオーバー**は、次のモデルに移行する前に、プロバイダー内部で発生します。

関連項目：

-   `agents.defaults.models` は、OpenClawが使用できるモデル（およびエイリアス）の許可リスト/カタログです。
-   `agents.defaults.imageModel` は、**プライマリモデルが画像を受け入れられない場合にのみ**使用されます。
-   エージェントごとのデフォルト設定は、`agents.list[].model` とバインディングを介して `agents.defaults.model` を上書きできます（[/concepts/multi-agent](./multi-agent.md) を参照）。

## モデルポリシーの概要

-   プライマリモデルは、利用可能な最新世代の最強モデルに設定してください。
-   コストやレイテンシに敏感なタスクや重要度の低いチャットにはフォールバックを使用してください。
-   ツール対応エージェントや信頼できない入力の場合、古い/弱いモデル階層は避けてください。

## セットアップウィザード（推奨）

設定を手動で編集したくない場合は、オンボーディングウィザードを実行してください：

```bash
openclaw onboard
```

これにより、一般的なプロバイダー（**OpenAI Code (Codex) サブスクリプション** (OAuth) や **Anthropic** (APIキーまたは `claude setup-token`) を含む）のモデルと認証を設定できます。

## 設定キー（概要）

-   `agents.defaults.model.primary` および `agents.defaults.model.fallbacks`
-   `agents.defaults.imageModel.primary` および `agents.defaults.imageModel.fallbacks`
-   `agents.defaults.models` (許可リスト + エイリアス + プロバイダーパラメータ)
-   `models.providers` (`models.json` に書き込まれるカスタムプロバイダー)

モデル参照は小文字に正規化されます。`z.ai/*` のようなプロバイダーエイリアスは `zai/*` に正規化されます。プロバイダー設定の例（OpenCode Zenを含む）は、[/gateway/configuration](../gateway/configuration.md#opencode-zen-multi-model-proxy) にあります。

## 「モデルは許可されていません」（および返信が停止する理由）

`agents.defaults.models` が設定されている場合、それは `/model` およびセッションオーバーライドのための**許可リスト**になります。ユーザーがその許可リストにないモデルを選択すると、OpenClawは以下を返します：

```bash
Model "provider/model" is not allowed. Use /model to list available models.
```

これは通常の返信が生成される**前に**発生するため、メッセージが「応答しなかった」ように感じられることがあります。修正方法は以下のいずれかです：

-   モデルを `agents.defaults.models` に追加する、または
-   許可リストをクリアする（`agents.defaults.models` を削除する）、または
-   `/model list` からモデルを選択する。

許可リスト設定の例：

```json
{
  agent: {
    model: { primary: "anthropic/claude-sonnet-4-5" },
    models: {
      "anthropic/claude-sonnet-4-5": { alias: "Sonnet" },
      "anthropic/claude-opus-4-6": { alias: "Opus" },
    },
  },
}
```

## チャットでのモデル切り替え (/model)

現在のセッションのモデルを再起動せずに切り替えることができます：

```
/model
/model list
/model 3
/model openai/gpt-5.2
/model status
```

注意点：

-   `/model`（および `/model list`）は、コンパクトな番号付きピッカー（モデルファミリー + 利用可能なプロバイダー）です。
-   Discordでは、`/model` と `/models` は、プロバイダーとモデルのドロップダウン、および送信ステップを含むインタラクティブなピッカーを開きます。
-   `/model <#>` はそのピッカーから選択します。
-   `/model status` は詳細ビューです（認証候補、および設定されている場合はプロバイダーエンドポイント `baseUrl` + `api` モード）。
-   モデル参照は、**最初の** `/` で分割して解析されます。`/model ` と入力する際は `provider/model` を使用してください。
-   モデルID自体に `/` が含まれている場合（OpenRouterスタイル）、プロバイダープレフィックスを含める必要があります（例：`/model openrouter/moonshotai/kimi-k2`）。
-   プロバイダーを省略した場合、OpenClawは入力をエイリアスまたは**デフォルトプロバイダー**のモデルとして扱います（モデルIDに `/` が含まれていない場合のみ機能します）。

完全なコマンドの動作/設定：[スラッシュコマンド](../tools/slash-commands.md)。

## CLIコマンド

```bash
openclaw models list
openclaw models status
openclaw models set <provider/model>
openclaw models set-image <provider/model>

openclaw models aliases list
openclaw models aliases add <alias> <provider/model>
openclaw models aliases remove <alias>

openclaw models fallbacks list
openclaw models fallbacks add <provider/model>
openclaw models fallbacks remove <provider/model>
openclaw models fallbacks clear

openclaw models image-fallbacks list
openclaw models image-fallbacks add <provider/model>
openclaw models image-fallbacks remove <provider/model>
openclaw models image-fallbacks clear
```

`openclaw models`（サブコマンドなし）は `models status` のショートカットです。

### models list

デフォルトでは設定済みモデルを表示します。便利なフラグ：

-   `--all`: 完全なカタログ
-   `--local`: ローカルプロバイダーのみ
-   `--provider `: プロバイダーでフィルタリング
-   `--plain`: 1行に1モデル
-   `--json`: 機械可読な出力

### models status

解決されたプライマリモデル、フォールバック、画像モデル、および設定済みプロバイダーの認証概要を表示します。また、認証ストアで見つかったプロファイルのOAuth有効期限ステータスを表示します（デフォルトでは24時間以内に警告）。`--plain` は解決されたプライマリモデルのみを出力します。OAuthステータスは常に表示されます（`--json` 出力にも含まれます）。設定済みプロバイダーに認証情報がない場合、`models status` は **Missing auth** セクションを出力します。JSONには `auth.oauth`（警告ウィンドウ + プロファイル）と `auth.providers`（プロバイダーごとの有効な認証）が含まれます。自動化には `--check` を使用してください（認証情報がない/期限切れの場合は終了コード `1`、期限切れ間近の場合は `2`）。認証の選択はプロバイダー/アカウントに依存します。常時稼働ゲートウェイホストの場合、APIキーが通常最も予測可能です。サブスクリプショントークンフローもサポートされています。例（Anthropic setup-token）：

```bash
claude setup-token
openclaw models status
```

## スキャン（OpenRouter 無料モデル）

`openclaw models scan` は、OpenRouterの**無料モデルカタログ**を検査し、オプションでモデルのツールおよび画像サポートをプローブできます。主なフラグ：

-   `--no-probe`: ライブプローブをスキップ（メタデータのみ）
-   `--min-params `: 最小パラメータサイズ（10億単位）
-   `--max-age-days `: 古いモデルをスキップ
-   `--provider `: プロバイダープレフィックスフィルター
-   `--max-candidates `: フォールバックリストのサイズ
-   `--set-default`: `agents.defaults.model.primary` を最初の選択肢に設定
-   `--set-image`: `agents.defaults.imageModel.primary` を最初の画像選択肢に設定

プローブにはOpenRouter APIキー（認証プロファイルまたは `OPENROUTER_API_KEY` から）が必要です。キーがない場合は、`--no-probe` を使用して候補のみをリストします。スキャン結果は以下の基準でランク付けされます：

1.  画像サポート
2.  ツールレイテンシ
3.  コンテキストサイズ
4.  パラメータ数

入力

-   OpenRouter `/models` リスト（フィルター `:free`）
-   認証プロファイルまたは `OPENROUTER_API_KEY` からのOpenRouter APIキーが必要（[/environment](../help/environment.md) を参照）
-   オプションフィルター：`--max-age-days`、`--min-params`、`--provider`、`--max-candidates`
-   プローブ制御：`--timeout`、`--concurrency`

TTYで実行すると、フォールバックをインタラクティブに選択できます。非対話モードでは、デフォルトを受け入れるために `--yes` を渡します。

## モデルレジストリ (models.json)

`models.providers` 内のカスタムプロバイダーは、エージェントディレクトリ（デフォルト `~/.openclaw/agents//models.json`）の下の `models.json` に書き込まれます。このファイルは、`models.mode` が `replace` に設定されていない限り、デフォルトでマージされます。一致するプロバイダーIDに対するマージモードの優先順位：

-   エージェント `models.json` に既に存在する空でない `baseUrl` が優先されます。
-   エージェント `models.json` 内の空でない `apiKey` は、そのプロバイダーが現在の設定/認証プロファイルコンテキストでSecretRef管理されていない場合にのみ優先されます。
-   SecretRef管理プロバイダーの `apiKey` 値は、解決されたシークレットを永続化する代わりに、ソースマーカー（環境変数参照の場合は `ENV_VAR_NAME`、ファイル/実行参照の場合は `secretref-managed`）から更新されます。
-   空または存在しないエージェント `apiKey`/`baseUrl` は、設定 `models.providers` にフォールバックします。
-   その他のプロバイダーフィールドは、設定および正規化されたカタログデータから更新されます。

このマーカーベースの永続化は、`openclaw agent` などのコマンド駆動パスを含め、OpenClawが `models.json` を再生成するたびに適用されます。

[モデルプロバイダークイックスタート](../providers/models.md)[モデルプロバイダー](./model-providers.md)