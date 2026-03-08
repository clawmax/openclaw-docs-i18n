title: "OpenClaw トークン使用量とコスト リファレンスガイド"
description: "OpenClawがトークンを追跡し、コストを推定し、コンテキストを管理する方法を学びましょう。使用量を監視するコマンドとトークン消費を削減する戦略を発見してください。"
keywords: ["openclaw トークン", "トークン使用量", "コスト推定", "コンテキストウィンドウ", "プロンプトキャッシング", "キャッシュ ttl", "トークン使用量削減", "システムプロンプト"]
---

  技術リファレンス

  
# トークン使用量とコスト

OpenClawは**文字数**ではなく**トークン**を追跡します。トークンはモデル固有ですが、ほとんどのOpenAIスタイルのモデルでは英語テキストに対して平均〜4文字が1トークンに相当します。

## システムプロンプトの構築方法

OpenClawは毎回実行時に独自のシステムプロンプトを組み立てます。これには以下が含まれます：

-   ツールリスト + 短い説明
-   スキルリスト（メタデータのみ。指示は`read`でオンデマンドで読み込まれます）
-   自己更新指示
-   ワークスペース + ブートストラップファイル（`AGENTS.md`、`SOUL.md`、`TOOLS.md`、`IDENTITY.md`、`USER.md`、`HEARTBEAT.md`、新規時は`BOOTSTRAP.md`、さらに存在する場合は`MEMORY.md`および/または`memory.md`）。大きなファイルは`agents.defaults.bootstrapMaxChars`（デフォルト: 20000）で切り詰められ、ブートストラップ注入の合計は`agents.defaults.bootstrapTotalMaxChars`（デフォルト: 150000）で制限されます。`memory/*.md`ファイルはメモリツール経由でオンデマンドであり、自動注入されません。
-   時間（UTC + ユーザータイムゾーン）
-   返信タグ + ハートビート動作
-   ランタイムメタデータ（ホスト/OS/モデル/思考）

詳細な内訳は[システムプロンプト](../concepts/system-prompt.md)をご覧ください。

## コンテキストウィンドウにカウントされるもの

モデルが受け取るすべてのものがコンテキスト制限にカウントされます：

-   システムプロンプト（上記の全セクション）
-   会話履歴（ユーザー + アシスタントメッセージ）
-   ツール呼び出しとツール結果
-   添付ファイル/トランスクリプト（画像、音声、ファイル）
-   圧縮要約と剪定アーティファクト
-   プロバイダーラッパーや安全性ヘッダー（表示されませんが、カウントされます）

画像の場合、OpenClawはプロバイダー呼び出し前にトランスクリプト/ツール画像ペイロードをダウンスケールします。`agents.defaults.imageMaxDimensionPx`（デフォルト: `1200`）を使用してこれを調整できます：

-   低い値は通常、ビジョントークン使用量とペイロードサイズを削減します。
-   高い値はOCR/UIの多いスクリーンショットの視覚的詳細をより多く保持します。

実用的な内訳（注入ファイル、ツール、スキル、システムプロンプトサイズごと）については、`/context list`または`/context detail`を使用してください。[コンテキスト](../concepts/context.md)を参照してください。

## 現在のトークン使用量を確認する方法

チャットで以下を使用します：

-   `/status` → **絵文字豊富なステータスカード**に、セッションモデル、コンテキスト使用量、最後の応答の入力/出力トークン、および**推定コスト**（APIキーのみ）が表示されます。
-   `/usage off|tokens|full` → すべての返信に**応答ごとの使用量フッター**を追加します。
    -   セッションごとに永続化されます（`responseUsage`として保存）。
    -   OAuth認証では**コストは非表示**になります（トークンのみ）。
-   `/usage cost` → OpenClawセッションログからのローカルコスト概要を表示します。

その他のインターフェース：

-   **TUI/Web TUI:** `/status` + `/usage` がサポートされています。
-   **CLI:** `openclaw status --usage` および `openclaw channels list` はプロバイダーのクォータウィンドウを表示します（応答ごとのコストではありません）。

## コスト推定（表示される場合）

コストはモデル価格設定から推定されます：

```
models.providers.<provider>.models[].cost
```

これらは`input`、`output`、`cacheRead`、`cacheWrite`の**100万トークンあたりのUSD**です。価格設定が欠落している場合、OpenClawはトークンのみを表示します。OAuthトークンはドルコストを表示しません。

## キャッシュTTLと剪定の影響

プロバイダーのプロンプトキャッシングは、キャッシュTTLウィンドウ内でのみ適用されます。OpenClawはオプションで**キャッシュTTL剪定**を実行できます：キャッシュTTLが期限切れになったらセッションを剪定し、その後キャッシュウィンドウをリセットして、後続のリクエストが完全な履歴を再キャッシュする代わりに新しくキャッシュされたコンテキストを再利用できるようにします。これにより、セッションがTTLを超えてアイドル状態になった場合のキャッシュ書き込みコストを低く保ちます。[ゲートウェイ設定](../gateway/configuration.md)で設定し、動作の詳細は[セッション剪定](../concepts/session-pruning.md)を参照してください。ハートビートはアイドル期間をまたいでキャッシュを**ウォーム**に保つことができます。モデルのキャッシュTTLが`1h`の場合、ハートビート間隔をそれより少し短く（例：`55m`）設定すると、完全なプロンプトの再キャッシュを回避し、キャッシュ書き込みコストを削減できます。マルチエージェント設定では、1つの共有モデル設定を維持し、`agents.list[].params.cacheRetention`でエージェントごとにキャッシュ動作を調整できます。詳細な設定ガイドは[プロンプトキャッシング](./prompt-caching.md)を参照してください。Anthropic APIの価格設定では、キャッシュ読み取りは入力トークンよりも大幅に安価ですが、キャッシュ書き込みはより高い乗数で課金されます。最新のレートとTTL乗数については、Anthropicのプロンプトキャッシング価格設定を参照してください：[https://docs.anthropic.com/docs/build-with-claude/prompt-caching](https://docs.anthropic.com/docs/build-with-claude/prompt-caching)

### 例：ハートビートで1時間のキャッシュをウォームに保つ

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long"
    heartbeat:
      every: "55m"
```

### 例：エージェントごとのキャッシュ戦略による混合トラフィック

```yaml
agents:
  defaults:
    model:
      primary: "anthropic/claude-opus-4-6"
    models:
      "anthropic/claude-opus-4-6":
        params:
          cacheRetention: "long" # ほとんどのエージェントのデフォルトベースライン
  list:
    - id: "research"
      default: true
      heartbeat:
        every: "55m" # 深いセッションのために長いキャッシュをウォームに保つ
    - id: "alerts"
      params:
        cacheRetention: "none" # バースト的な通知のキャッシュ書き込みを回避
```

`agents.list[].params`は選択されたモデルの`params`の上にマージされるため、`cacheRetention`のみをオーバーライドし、他のモデルデフォルトは変更せずに継承できます。

### 例：Anthropic 1Mコンテキストベータヘッダーを有効化

Anthropicの1Mコンテキストウィンドウは現在ベータゲートされています。OpenClawは、サポートされているOpusまたはSonnetモデルで`context1m`を有効にすると、必要な`anthropic-beta`値を注入できます。

```yaml
agents:
  defaults:
    models:
      "anthropic/claude-opus-4-6":
        params:
          context1m: true
```

これはAnthropicの`context-1m-2025-08-07`ベータヘッダーにマッピングされます。これは`context1m: true`がそのモデルエントリに設定されている場合にのみ適用されます。要件：資格情報が長いコンテキスト使用の対象である必要があります（APIキー課金、またはExtra Usageが有効なサブスクリプション）。そうでない場合、Anthropicは`HTTP 429: rate_limit_error: Extra usage is required for long context requests`で応答します。AnthropicをOAuth/サブスクリプショントークン（`sk-ant-oat-*`）で認証する場合、OpenClawは`context-1m-*`ベータヘッダーをスキップします。なぜなら、Anthropicは現在その組み合わせをHTTP 401で拒否するためです。

## トークン負荷を軽減するためのヒント

-   長いセッションを要約するには`/compact`を使用します。
-   ワークフローで大きなツール出力をトリミングします。
-   スクリーンショットの多いセッションでは`agents.defaults.imageMaxDimensionPx`を下げます。
-   スキルの説明は短く保ちます（スキルリストはプロンプトに注入されます）。
-   冗長で探索的な作業には小さなモデルを優先します。

正確なスキルリストのオーバーヘッド計算式については[スキル](../tools/skills.md)を参照してください。

[ウィザードリファレンス](./wizard.md)[SecretRef 資格情報インターフェース](./secretref-credential-surface.md)

---