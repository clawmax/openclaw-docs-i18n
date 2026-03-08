

  技術リファレンス

  
# プロンプトキャッシング

プロンプトキャッシングとは、モデルプロバイダーが変更されていないプロンプトの接頭辞（通常はシステム/開発者指示やその他の安定したコンテキスト）を毎回再処理する代わりに、複数のターンにわたって再利用できることを意味します。最初に一致するリクエストがキャッシュトークンを書き込み（`cacheWrite`）、後続の一致するリクエストがそれを読み戻す（`cacheRead`）ことができます。これが重要な理由：トークンコストの削減、応答の高速化、長時間実行されるセッションでの予測可能なパフォーマンスです。キャッシングがない場合、繰り返されるプロンプトは、入力の大部分が変更されていなくても、毎ターン完全なプロンプトコストを支払うことになります。このページでは、プロンプトの再利用とトークンコストに影響するすべてのキャッシュ関連の設定項目をカバーします。Anthropicの料金詳細については、以下を参照してください：[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.thropic.com/docs/build-with-claude/prompt-caching)

## 主要な設定項目

### cacheRetention (モデルおよびエージェントごと)

モデルパラメータでキャッシュ保持を設定：

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "short" # none | short | long
```

エージェントごとのオーバーライド：

```yaml
agents:
  list:
    - id: "alerts"
      params:
        cacheRetention: "none"
```

設定のマージ順序：

1.  `agents.defaults.models["provider/model"].params`
2.  `agents.list[].params` (エージェントIDに一致；キーごとにオーバーライド)

### 従来の cacheControlTtl

従来の値も受け入れられ、マッピングされます：

-   `5m` -> `short`
-   `1h` -> `long`

新しい設定では`cacheRetention`を推奨します。

### contextPruning.mode: "cache-ttl"

キャッシュTTLウィンドウ後に古いツール結果のコンテキストを削除し、アイドル後のリクエストが過大な履歴を再キャッシュしないようにします。

```yaml
agents:
  defaults:
    contextPruning:
      mode: "cache-ttl"
      ttl: "1h"
```

完全な動作については[セッションプルーニング](../concepts/session-pruning.md)を参照してください。

### ハートビートによるウォームキープ

ハートビートはキャッシュウィンドウをウォームに保ち、アイドルギャップ後の繰り返しキャッシュ書き込みを減らすことができます。

```yaml
agents:
  defaults:
    heartbeat:
      every: "55m"
```

エージェントごとのハートビートは`agents.list[].heartbeat`でサポートされています。

## プロバイダーの動作

### Anthropic (直接API)

-   `cacheRetention`がサポートされています。
-   Anthropic APIキー認証プロファイルでは、OpenClawは設定されていないAnthropicモデル参照に対して`cacheRetention: "short"`をシードします。

### Amazon Bedrock

-   Anthropic Claudeモデル参照（`amazon-bedrock/*anthropic.claude*`）は、明示的な`cacheRetention`のパススルーをサポートします。
-   Anthropic以外のBedrockモデルは、実行時に`cacheRetention: "none"`に強制されます。

### OpenRouter Anthropicモデル

`openrouter/anthropic/*`モデル参照の場合、OpenClawはシステム/開発者プロンプトブロックにAnthropicの`cache_control`を注入し、プロンプトキャッシュの再利用を改善します。

### その他のプロバイダー

プロバイダーがこのキャッシュモードをサポートしていない場合、`cacheRetention`は効果がありません。

## チューニングパターン

### 混合トラフィック (推奨デフォルト)

メインエージェントで長寿命のベースラインを維持し、バースト的な通知エージェントではキャッシングを無効にします：

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m"
    - id: "alerts"
      params:
        cacheRetention: "none"
```

### コスト優先ベースライン

-   ベースライン`cacheRetention: "short"`を設定します。
-   `contextPruning.mode: "cache-ttl"`を有効にします。
-   ウォームキャッシュの恩恵を受けるエージェントに対してのみ、TTLを下回るハートビートを維持します。

## キャッシュ診断

OpenClawは、埋め込みエージェント実行用の専用キャッシュトレース診断を公開しています。

### diagnostics.cacheTrace 設定

```yaml
diagnostics:
  cacheTrace:
    enabled: true
    filePath: "~/.openclaw/logs/cache-trace.jsonl" # オプション
    includeMessages: false # デフォルト true
    includePrompt: false # デフォルト true
    includeSystem: false # デフォルト true
```

デフォルト：

-   `filePath`: `$OPENCLAW_STATE_DIR/logs/cache-trace.jsonl`
-   `includeMessages`: `true`
-   `includePrompt`: `true`
-   `includeSystem`: `true`

### 環境変数トグル (ワンオフデバッグ)

-   `OPENCLAW_CACHE_TRACE=1` キャッシュトレースを有効にします。
-   `OPENCLAW_CACHE_TRACE_FILE=/path/to/cache-trace.jsonl` 出力パスをオーバーライドします。
-   `OPENCLAW_CACHE_TRACE_MESSAGES=0|1` 完全なメッセージペイロードのキャプチャを切り替えます。
-   `OPENCLAW_CACHE_TRACE_PROMPT=0|1` プロンプトテキストのキャプチャを切り替えます。
-   `OPENCLAW_CACHE_TRACE_SYSTEM=0|1` システムプロンプトのキャプチャを切り替えます。

### 検査項目

-   キャッシュトレースイベントはJSONL形式で、`session:loaded`、`prompt:before`、`stream:context`、`session:after`などの段階的なスナップショットを含みます。
-   ターンごとのキャッシュトークンへの影響は、`cacheRead`と`cacheWrite`（例：`/usage full`やセッション使用量サマリー）を介して、通常の使用状況サーフェスで確認できます。

## クイックトラブルシューティング

-   ほとんどのターンで高い`cacheWrite`：揮発性のシステムプロンプト入力を確認し、モデル/プロバイダーがキャッシュ設定をサポートしていることを確認してください。
-   `cacheRetention`の効果がない：モデルキーが`agents.defaults.models["provider/model"]`と一致していることを確認してください。
-   キャッシュ設定付きのBedrock Nova/Mistralリクエスト：実行時に`none`に強制されることが期待されます。

関連ドキュメント：

-   [Anthropic](../providers/anthropic.md)
-   [トークン使用とコスト](./token-use.md)
-   [セッションプルーニング](../concepts/session-pruning.md)
-   [ゲートウェイ設定リファレンス](../gateway/configuration-reference.md)

[SecretRef 認証情報サーフェス](./secretref-credential-surface.md)[API使用とコスト](./api-usage-costs.md)