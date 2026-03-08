

  セッションとメモリ

  
# メモリ

OpenClawのメモリは、**エージェントワークスペース内のプレーンなMarkdownファイル**です。ファイルが信頼できる情報源であり、モデルが「記憶」するのはディスクに書き込まれた内容のみです。メモリ検索ツールはアクティブなメモリプラグイン（デフォルト: `memory-core`）によって提供されます。メモリプラグインは `plugins.slots.memory = "none"` で無効化できます。

## メモリファイル (Markdown)

デフォルトのワークスペースレイアウトでは、2つのメモリレイヤーを使用します:

-   `memory/YYYY-MM-DD.md`
    -   日次ログ（追記のみ）。
    -   セッション開始時に今日と昨日のファイルを読み込みます。
-   `MEMORY.md` (オプション)
    -   精選された長期記憶。
    -   **メインのプライベートセッションでのみ読み込まれます**（グループコンテキストでは決して読み込まれません）。

これらのファイルはワークスペース（`agents.defaults.workspace`、デフォルトは `~/.openclaw/workspace`）の下に存在します。完全なレイアウトについては [エージェントワークスペース](./agent-workspace.md) を参照してください。

## メモリツール

OpenClawはこれらのMarkdownファイルに対して、エージェント向けに2つのツールを公開しています:

-   `memory_search` — インデックスされたスニペットに対するセマンティックな検索。
-   `memory_get` — 特定のMarkdownファイル/行範囲を対象とした読み取り。

`memory_get` は、ファイルが存在しない場合（例えば、初回書き込み前の今日の日次ログ）に **適切に機能を低下させます**。組み込みマネージャーとQMDバックエンドの両方が `ENOENT` をスローする代わりに `{ text: "", path }` を返すため、エージェントは「まだ何も記録されていない」状況を処理し、ツール呼び出しをtry/catchロジックでラップすることなくワークフローを継続できます。

## メモリを書き込むタイミング

-   決定事項、好み、永続的な事実は `MEMORY.md` に書き込みます。
-   日々のメモや進行中のコンテキストは `memory/YYYY-MM-DD.md` に書き込みます。
-   誰かが「これを覚えておいて」と言ったら、書き留めてください（RAMに保持しないでください）。
-   この領域はまだ進化中です。モデルに記憶を保存するよう促すことが役立ちます。モデルは何をすべきか理解します。
-   何かを確実に記憶させたい場合は、**ボットにメモリに書き込むよう依頼してください**。

## 自動メモリフラッシュ（事前コンパクション通知）

セッションが **自動コンパクションに近づいた** とき、OpenClawは **サイレントなエージェントターン** をトリガーし、コンテキストが圧縮される **前に** モデルに永続的なメモリを書き込むよう促します。デフォルトのプロンプトは、モデルが *返信してもよい* と明示していますが、通常はユーザーにこのターンが見えないように `NO_REPLY` が正しい応答です。これは `agents.defaults.compaction.memoryFlush` で制御されます:

```json
{
  agents: {
    defaults: {
      compaction: {
        reserveTokensFloor: 20000,
        memoryFlush: {
          enabled: true,
          softThresholdTokens: 4000,
          systemPrompt: "セッションがコンパクションに近づいています。永続的なメモリを今すぐ保存してください。",
          prompt: "永続的なメモを memory/YYYY-MM-DD.md に書き込んでください。保存するものがなければ NO_REPLY で返信してください。",
        },
      },
    },
  },
}
```

詳細:

-   **ソフトしきい値**: セッションのトークン推定値が `contextWindow - reserveTokensFloor - softThresholdTokens` を超えたときにフラッシュがトリガーされます。
-   デフォルトで **サイレント**: プロンプトには `NO_REPLY` が含まれるため、何も配信されません。
-   **2つのプロンプト**: ユーザープロンプトに加え、システムプロンプトがリマインダーを追加します。
-   **コンパクションサイクルごとに1回のフラッシュ** (`sessions.json` で追跡)。
-   **ワークスペースは書き込み可能である必要があります**: セッションが `workspaceAccess: "ro"` または `"none"` でサンドボックス化されている場合、フラッシュはスキップされます。

完全なコンパクションのライフサイクルについては、[セッション管理 + コンパクション](../reference/session-management-compaction.md) を参照してください。

## ベクターメモリ検索

OpenClawは `MEMORY.md` と `memory/*.md` に対して小さなベクターインデックスを構築でき、セマンティッククエリで表現が異なる場合でも関連するメモを見つけることができます。デフォルト:

-   デフォルトで有効。
-   メモリファイルの変更を監視（デバウンス）。
-   メモリ検索は `agents.defaults.memorySearch` で設定（トップレベルの `memorySearch` ではありません）。
-   デフォルトでリモート埋め込みを使用。`memorySearch.provider` が設定されていない場合、OpenClawは自動選択します:
    1.  `memorySearch.local.modelPath` が設定され、ファイルが存在する場合は `local`。
    2.  OpenAIキーが解決できる場合は `openai`。
    3.  Geminiキーが解決できる場合は `gemini`。
    4.  Voyageキーが解決できる場合は `voyage`。
    5.  Mistralキーが解決できる場合は `mistral`。
    6.  それ以外の場合は、設定されるまでメモリ検索は無効のまま。
-   ローカルモードは node-llama-cpp を使用し、`pnpm approve-builds` が必要な場合があります。
-   sqlite-vec（利用可能な場合）を使用して、SQLite内でのベクター検索を高速化。
-   `memorySearch.provider = "ollama"` も、ローカル/セルフホストされたOllama埋め込み（`/api/embeddings`）でサポートされていますが、自動選択はされません。

リモート埋め込みには、埋め込みプロバイダーのAPIキーが **必須** です。OpenClawは認証プロファイル、`models.providers.*.apiKey`、または環境変数からキーを解決します。Codex OAuthはチャット/補完のみをカバーし、メモリ検索の埋め込みには **対応していません**。Geminiの場合は `GEMINI_API_KEY` または `models.providers.google.apiKey` を使用してください。Voyageの場合は `VOYAGE_API_KEY` または `models.providers.voyage.apiKey` を使用してください。Mistralの場合は `MISTRAL_API_KEY` または `models.providers.mistral.apiKey` を使用してください。Ollamaは通常、実際のAPIキーを必要としません（ローカルポリシーで必要な場合は `OLLAMA_API_KEY=ollama-local` のようなプレースホルダーで十分です）。カスタムのOpenAI互換エンドポイントを使用する場合は、`memorySearch.remote.apiKey`（およびオプションで `memorySearch.remote.headers`）を設定してください。

### QMDバックエンド（実験的）

`memory.backend = "qmd"` を設定すると、組み込みのSQLiteインデクサーを [QMD](https://github.com/tobi/qmd) に置き換えます: BM25 + ベクター + 再ランキングを組み合わせたローカルファーストの検索サイドカーです。Markdownは信頼できる情報源のままです。OpenClawは検索のためにQMDをシェルアウトします。重要なポイント: **前提条件**

-   デフォルトでは無効。設定ごとにオプトイン（`memory.backend = "qmd"`）。
-   QMD CLIを別途インストール（`bun install -g https://github.com/tobi/qmd` またはリリースを取得）し、`qmd` バイナリがゲートウェイの `PATH` 上にあることを確認。
-   QMDは拡張機能を許可するSQLiteビルドが必要（macOSでは `brew install sqlite`）。
-   QMDはBun + `node-llama-cpp` 経由で完全にローカルで実行され、初回使用時にHuggingFaceからGGUFモデルを自動ダウンロード（別途Ollamaデーモンは不要）。
-   ゲートウェイは `XDG_CONFIG_HOME` と `XDG_CACHE_HOME` を設定することで、`~/.openclaw/agents//qmd/` 下の自己完結型XDGホームでQMDを実行。
-   OSサポート: macOSとLinuxは、Bun + SQLiteがインストールされていればすぐに動作。WindowsはWSL2経由でのサポートが最適。

**サイドカーの実行方法**

-   ゲートウェイは `~/.openclaw/agents//qmd/` 下に自己完結型QMDホームを書き込み（設定 + キャッシュ + sqlite DB）。
-   コレクションは `memory.qmd.paths`（デフォルトのワークスペースメモリファイルに加えて）から `qmd collection add` で作成され、その後 `qmd update` + `qmd embed` が起動時と設定可能な間隔（`memory.qmd.update.interval`、デフォルト5分）で実行。
-   ゲートウェイは起動時にQMDマネージャーを初期化するようになったため、最初の `memory_search` 呼び出しの前でも定期的な更新タイマーがセットされます。
-   起動時の更新はデフォルトでバックグラウンドで実行されるため、チャットの起動がブロックされません。以前のブロッキング動作を維持するには `memory.qmd.update.waitForBootSync = true` を設定。
-   検索は `memory.qmd.searchMode` 経由で実行（デフォルト `qmd search --json`; `vsearch` と `query` もサポート）。選択したモードが使用中のQMDビルドでフラグを拒否する場合、OpenClawは `qmd query` で再試行します。QMDが失敗するかバイナリが見つからない場合、OpenClawは自動的に組み込みのSQLiteマネージャーにフォールバックし、メモリツールが動作し続けるようにします。
-   OpenClawは現在、QMDの埋め込みバッチサイズのチューニングを公開していません。バッチ動作はQMD自体によって制御されます。
-   **最初の検索は遅くなる可能性があります**: QMDは最初の `qmd query` 実行時にローカルGGUFモデル（再ランカー/クエリ拡張）をダウンロードする場合があります。
    -   OpenClawはQMDを実行するときに自動的に `XDG_CONFIG_HOME`/`XDG_CACHE_HOME` を設定します。
    -   モデルを手動で事前ダウンロードし（かつOpenClawが使用するのと同じインデックスをウォームアップしたい場合）、エージェントのXDGディレクトリでワンオフクエリを実行します。OpenClawのQMD状態は **状態ディレクトリ**（デフォルト `~/.openclaw`）の下にあります。OpenClawが使用するのと同じXDG変数をエクスポートすることで、`qmd` を正確に同じインデックスに向けることができます:
        
        Copy
        
        ```bash
        # OpenClawが使用するのと同じ状態ディレクトリを選択
        STATE_DIR="${OPENCLAW_STATE_DIR:-$HOME/.openclaw}"
        
        export XDG_CONFIG_HOME="$STATE_DIR/agents/main/qmd/xdg-config"
        export XDG_CACHE_HOME="$STATE_DIR/agents/main/qmd/xdg-cache"
        
        # (オプション) インデックス更新 + 埋め込みを強制実行
        qmd update
        qmd embed
        
        # ウォームアップ / 初回モデルダウンロードをトリガー
        qmd query "test" -c memory-root --json >/dev/null 2>&1
        ```
        

**設定項目 (`memory.qmd.*`)**

-   `command` (デフォルト `qmd`): 実行可能ファイルのパスを上書き。
-   `searchMode` (デフォルト `search`): `memory_search` を支えるQMDコマンドを選択（`search`, `vsearch`, `query`）。
-   `includeDefaultMemory` (デフォルト `true`): `MEMORY.md` + `memory/**/*.md` を自動インデックス。
-   `paths[]`: 追加のディレクトリ/ファイルを追加（`path`、オプションの `pattern`、オプションの安定した `name`）。
-   `sessions`: セッションJSONLインデックス化をオプトイン（`enabled`, `retentionDays`, `exportDir`）。
-   `update`: 更新頻度とメンテナンス実行を制御: (`interval`, `debounceMs`, `onBoot`, `waitForBootSync`, `embedInterval`, `commandTimeoutMs`, `updateTimeoutMs`, `embedTimeoutMs`)。
-   `limits`: 検索結果のペイロードを制限（`maxResults`, `maxSnippetChars`, `maxInjectedChars`, `timeoutMs`）。
-   `scope`: [`session.sendPolicy`](../gateway/configuration.md#session) と同じスキーマ。デフォルトはDMのみ（すべて `deny`、`allow` ダイレクトチャット）。グループ/チャンネルでQMDヒットを表示するには緩和。
    -   `match.keyPrefix` は **正規化された** セッションキー（小文字化、先頭の `agent::` を除去）にマッチ。例: `discord:channel:`。
    -   `match.rawKeyPrefix` は **生の** セッションキー（小文字化）、`agent::` を含むものにマッチ。例: `agent:main:discord:`。
    -   レガシー: `match.keyPrefix: "agent:..."` は依然として生キープレフィックスとして扱われますが、明確さのために `rawKeyPrefix` を推奨。
-   `scope` が検索を拒否した場合、OpenClawは派生した `channel`/`chatType` を含む警告をログに記録し、空の結果のデバッグを容易にします。
-   ワークスペース外から取得したスニペットは、`memory_search` の結果で `qmd//<relative-path>` として表示されます。`memory_get` はそのプレフィックスを理解し、設定されたQMDコレクションルートから読み取ります。
-   `memory.qmd.sessions.enabled = true` の場合、OpenClawはサニタイズされたセッショントランスクリプト（User/Assistantターン）を `~/.openclaw/agents//qmd/sessions/` 下の専用QMDコレクションにエクスポートし、`memory_search` が組み込みSQLiteインデックスに触れることなく最近の会話を思い出せるようにします。
-   `memory_search` スニペットには、`memory.citations` が `auto`/`on` の場合、`Source: <path#line>` フッターが含まれるようになりました。`memory.citations = "off"` を設定すると、パスメタデータを内部に保持します（エージェントは `memory_get` 用のパスを受け取りますが、スニペットテキストからフッターは省略され、システムプロンプトはエージェントにそれを引用しないよう警告します）。

**例**

```
memory: {
  backend: "qmd",
  citations: "auto",
  qmd: {
    includeDefaultMemory: true,
    update: { interval: "5m", debounceMs: 15000 },
    limits: { maxResults: 6, timeoutMs: 4000 },
    scope: {
      default: "deny",
      rules: [
        { action: "allow", match: { chatType: "direct" } },
        // 正規化されたセッションキープレフィックス (`agent:<id>:` を除去)。
        { action: "deny", match: { keyPrefix: "discord:channel:" } },
        // 生のセッションキープレフィックス (`agent:<id>:` を含む)。
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ]
    },
    paths: [
      { name: "docs", path: "~/notes", pattern: "**/*.md" }
    ]
  }
}
```

**引用とフォールバック**

-   `memory.citations` はバックエンドに関係なく適用されます（`auto`/`on`/`off`）。
-   `qmd` が実行されるとき、`status().backend = "qmd"` をタグ付けし、診断にどのエンジンが結果を提供したかを表示します。QMDサブプロセスが終了するかJSON出力を解析できない場合、検索マネージャーは警告をログに記録し、QMDが回復するまで組み込みプロバイダー（既存のMarkdown埋め込み）を返します。

### 追加のメモリパス

デフォルトのワークスペースレイアウト外のMarkdownファイルをインデックス化したい場合は、明示的なパスを追加します:

```
agents: {
  defaults: {
    memorySearch: {
      extraPaths: ["../team-docs", "/srv/shared-notes/overview.md"]
    }
  }
}
```

注意点:

-   パスは絶対パスまたはワークスペース相対パスで指定できます。
-   ディレクトリは再帰的に `.md` ファイルをスキャンします。
-   Markdownファイルのみがインデックス化されます。
-   シンボリックリンクは無視されます（ファイルまたはディレクトリ）。

### Gemini埋め込み（ネイティブ）

プロバイダーを `gemini` に設定して、Gemini埋め込みAPIを直接使用します:

```
agents: {
  defaults: {
    memorySearch: {
      provider: "gemini",
      model: "gemini-embedding-001",
      remote: {
        apiKey: "YOUR_GEMINI_API_KEY"
      }
    }
  }
}
```

注意点:

-   `remote.baseUrl` はオプション（デフォルトはGemini APIベースURL）。
-   `remote.headers` は必要に応じて追加ヘッダーを追加できます。
-   デフォルトモデル: `gemini-embedding-001`。

**カスタムのOpenAI互換エンドポイント**（OpenRouter、vLLM、またはプロキシ）を使用したい場合は、OpenAIプロバイダーで `remote` 設定を使用できます:

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      remote: {
        baseUrl: "https://api.example.com/v1/",
        apiKey: "YOUR_OPENAI_COMPAT_API_KEY",
        headers: { "X-Custom-Header": "value" }
      }
    }
  }
}
```

APIキーを設定したくない場合は、`memorySearch.provider = "local"` を使用するか、`memorySearch.fallback = "none"` を設定します。フォールバック:

-   `memorySearch.fallback` は `openai`、`gemini`、`voyage`、`mistral`、`ollama`、`local`、`none` のいずれか。
-   フォールバックプロバイダーは、プライマリ埋め込みプロバイダーが失敗した場合にのみ使用されます。

バッチインデックス化（OpenAI + Gemini + Voyage）:

-   デフォルトでは無効。大規模コーパスのインデックス化（OpenAI、Gemini、Voyage）には `agents.defaults.memorySearch.remote.batch.enabled = true` を設定して有効化。
-   デフォルトの動作はバッチ完了を待機します。必要に応じて `remote.batch.wait`、`remote.batch.pollIntervalMs`、`remote.batch.timeoutMinutes` を調整。
-   `remote.batch.concurrency` を設定して、並行して送信するバッチジョブの数を制御（デフォルト: 2）。
-   バッチモードは `memorySearch.provider = "openai"` または `"gemini"` の場合に適用され、対応するAPIキーを使用。
-   Geminiバッチジョブは非同期埋め込みバッチエンドポイントを使用し、Gemini Batch APIの可用性が必要。

OpenAIバッチが高速で安価な理由:

-   大規模なバックフィルでは、OpenAIは通常、多くの埋め込みリクエストを単一のバッチジョブとして送信し、OpenAIに非同期で処理させることができるため、サポートしている中で最も高速なオプションです。
-   OpenAIはBatch APIワークロードに対して割引価格を提供するため、同じリクエストを同期的に送信するよりも大規模なインデックス化実行は通常安価です。
-   詳細はOpenAI Batch APIドキュメントと価格を参照:
    -   [https://platform.openai.com/docs/api-reference/batch](https://platform.openai.com/docs/api-reference/batch)
    -   [https://platform.openai.com/pricing](https://platform.openai.com/pricing)

設定例:

```
agents: {
  defaults: {
    memorySearch: {
      provider: "openai",
      model: "text-embedding-3-small",
      fallback: "openai",
      remote: {
        batch: { enabled: true, concurrency: 2 }
      },
      sync: { watch: true }
    }
  }
}
```

ツール:

-   `memory_search` — ファイル + 行範囲を含むスニペットを返します。
-   `memory_get` — パスでメモリファイルの内容を読み取ります。

ローカルモード:

-   `agents.defaults.memorySearch.provider = "local"` を設定。
-   `agents.defaults.memorySearch.local.modelPath`（GGUFまたは `hf:` URI）を提供。
-   オプション: `agents.defaults.memorySearch.fallback = "none"` を設定してリモートフォールバックを回避。

### メモリツールの仕組み

-   `memory_search` は `MEMORY.md` + `memory/**/*.md` からMarkdownチャンク（約400トークン目標、80トークンオーバーラップ）をセマンティックに検索します。スニペットテキスト（約700文字に制限）、ファイルパス、行範囲、スコア、プロバイダー/モデル、およびローカル→リモート埋め込みからフォールバックしたかどうかを返します。完全なファイルペイロードは返されません。
-   `memory_get` は特定のメモリMarkdownファイル（ワークスペース相対）を、オプションで開始行からN行読み取ります。`MEMORY.md` / `memory/` 外のパスは拒否されます。
-   両方のツールは、エージェントに対して `memorySearch.enabled` がtrueと解決された場合にのみ有効になります。

### 何がインデックス化されるか（およびいつ）

-   ファイルタイプ: Markdownのみ（`MEMORY.md`、`memory/**/*.md`）。
-   インデックスストレージ: エージェントごとのSQLite、`~/.openclaw/memory/.sqlite`（`agents.defaults.memorySearch.store.path` で設定可能、`{agentId}` トークンをサポート）。
-   鮮度: `MEMORY.md` + `memory/` のウォッチャーがインデックスをダーティとしてマーク（デバウンス1.5秒）。同期はセッション開始時、検索時、または間隔でスケジュールされ、非同期に実行されます。セッショントランスクリプトは差分しきい値を使用してバックグラウンド同期をトリガーします。
-   再インデックストリガー: インデックスは埋め込み **プロバイダー/モデル + エンドポイントフィンガープリント + チャンキングパラメータ** を保存します。これらのいずれかが変更されると、OpenClawは自動的にストア全体をリセットして再インデックス化します。

### ハイブリッド検索（BM25 + ベクター）

有効にすると、OpenClawは以下を組み合わせます:

-   **ベクター類似性**（セマンティックマッチ、表現が異なっても可）
-   **BM25キーワード関連性**（ID、環境変数、コードシンボルなどの正確なトークン）

フルテキスト検索がプラットフォームで利用できない場合、OpenClawはベクターのみの検索にフォールバックします。

#### ハイブリッド検索の理由

ベクター検索は「これと同じ意味」に優れています:

-   「Mac Studioゲートウェイホスト」 vs 「ゲートウェイを実行しているマシン」
-   「ファイル更新のデバウンス」 vs 「書き込みごとのインデックス化を避ける」

しかし、正確で高シグナルのトークンには弱い場合があります:

-   ID（`a828e60`、`b3b9895a…`）
-   コードシンボル（`memorySearch.query.hybrid`）
-   エラー文字列（「sqlite-vec unavailable」）

BM25（フルテキスト）はその逆です: 正確なトークンに強く、言い換えには弱い。ハイブリッド検索は実用的な中間地点です: **両方の検索シグナルを使用** して、「自然言語」クエリと「干し草の山の中の針」クエリの両方に対して良い結果を得られます。

#### 結果のマージ方法（現在の設計）

実装の概要:

1.  両側から候補プールを取得:

-   **ベクター**: コサイン類似度による上位 `maxResults * candidateMultiplier`。
-   **BM25**: FTS5 BM25ランクによる上位 `maxResults * candidateMultiplier`（低いほど良い）。

2.  BM25ランクを0..1に近いスコアに変換:

-   `textScore = 1 / (1 + max(0, bm25Rank))`

3.  チャンクIDで候補を統合し、加重スコアを計算:

-   `finalScore = vectorWeight * vectorScore + textWeight * textScore`

注意点:

-   `vectorWeight` + `textWeight` は設定解決時に1.0に正規化されるため、重みはパーセンテージとして動作します。
-   埋め込みが利用できない場合（またはプロバイダーがゼロベクターを返す場合）、BM25を実行してキーワードマッチを返します。
-   FTS5を作成できない場合、ベクターのみの検索を維持します（ハード失敗なし）。

これは「IR理論的に完璧」ではありませんが、シンプルで高速であり、実際のメモでの再現率/精度を向上させる傾向があります。後でより洗練させたい場合、一般的な次のステップは、マージ前の相互順位融合（RRF）またはスコア正規化（最小/最大またはzスコア）です。

#### 後処理パイプライン

ベクターとキーワードのスコアをマージした後、結果リストがエージェントに到達する前に、2つのオプションの後処理ステージで洗練されます:

```bash
ベクター + キーワード → 加重マージ → 時間減衰 → ソート → MMR → トップK結果
```

両方のステージは **デフォルトでオフ** であり、独立して有効化できます。

#### MMR再ランキング（多様性）

ハイブリッド検索が結果を返すとき、複数のチャンクに類似または重複するコンテンツが含まれる場合があります。例えば、「ホームネットワーク設定」を検索すると、同じルーター設定について言及している異なる日次メモから、ほぼ同一の5つのスニペットが返されるかもしれません。**MMR（Maximal Marginal Relevance）** は結果を再ランク付けし、関連性と多様性のバランスを取り、トップ結果が同じ情報を繰り返すのではなく、クエリの異なる側面をカバーするようにします。仕組み:

1.  結果は元の関連性（ベクター + BM25加重スコア）でスコアリングされます。
2.  MMRは反復的に、以下を最大化する結果を選択します: `λ × 関連性 − (1−λ) × max_similarity_to_selected`。
3.  結果間の類似性は、トークン化されたコンテンツに対するJaccardテキスト類似度で測定されます。

`lambda` パラメータはトレードオフを制御します:

-   `lambda = 1.0` → 純粋な関連性（多様性ペナルティなし）
-   `lambda = 0.0` → 最大多様性（関連性を無視）
-   デフォルト: `0.7`（バランス型、関連性にわずかに偏り）

**例 — クエリ: 「ホームネットワーク設定」** 以下のメモリファイルがある場合:

```bash
memory/2026-02-10.md  → "Omadaルーターを設定、IoTデバイスにVLAN 10を設定"
memory/2026-02-08.md  → "Omadaルーターを設定、IoTをVLAN 10に移動"
memory/2026-02-05.md  → "AdGuard DNSを192.168.10.2に設定"
memory/network.md     → "ルーター: Omada ER605, AdGuard: 192.168.10.2, VLAN 10: IoT"
```

MMRなし — トップ3結果:

```bash
1. memory/2026-02-10.md  (スコア: 0.92)  ← ルーター + VLAN
2. memory/2026-02-08.md  (スコア: 0.89)  ← ルーター + VLAN (ほぼ重複!)
3. memory/network.md     (スコア: 0.85)  ← 参照ドキュメント
```

MMRあり（λ=0.7） — トップ3結果:

```bash
1. memory/2026-02-10.md  (スコア: 0.92)  ← ルーター + VLAN
2. memory/network.md     (スコア: 0.85)  ← 参照ドキュメント (多様!)
3. memory/2026-02-05.md  (スコア: 0.78)  ← AdGuard DNS (多様!)
```

2月8日のほぼ重複が除外され、エージェントは3つの異なる情報を得ます。**有効化するタイミング:** `memory_search` が冗長またはほぼ重複するスニペットを返すことに気づいた場合、特に日次メモで日をまたいで類似情報が繰り返される場合。

#### 時間減衰（新規性ブースト）

日次メモを持つエージェントは、時間とともに数百の日付付きファイルを蓄積します。減衰がないと、6ヶ月前のうまく書かれたメモが、同じトピックに関する昨日の更新を上回ることがあります。**時間減衰** は、各結果の経過日数に基づいてスコアに指数乗数を適用し、最近のメモリが自然に上位にランクされ、古いものはフェードアウトします:

```bash
減衰スコア = スコア × e^(-λ × 経過日数)
```

ここで `λ = ln(2) / halfLifeDays`。デフォルトの半減期30日の場合:

-   今日のメモ: 元のスコアの **100%**
-   7日前: **~84%**
-   30日前: **50%**
-   90日前: **12.5%**
-   180日前: **~1.6%**

**常緑ファイルは減衰されません:**

-   `MEMORY.md`（ルートメモリファイル）
-   `memory/` 内の日付なしファイル（例: `memory/projects.md`、`memory/