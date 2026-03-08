

  プロバイダー

  
# MiniMax

MiniMaxは、**M2/M2.5**モデルファミリーを構築するAI企業です。現在のコーディングに焦点を当てたリリースは、実世界の複雑なタスクのために構築された**MiniMax M2.5**（2025年12月23日）です。出典: [MiniMax M2.5 リリースノート](https://www.minimax.io/news/minimax-m25)

## モデル概要 (M2.5)

MiniMaxはM2.5における以下の改善点を強調しています:

-   より強力な**多言語コーディング**（Rust、Java、Go、C++、Kotlin、Objective-C、TS/JS）。
-   より優れた**Web/アプリ開発**と美的出力品質（ネイティブモバイルを含む）。
-   インタリーブ思考と統合制約実行に基づく、オフィススタイルのワークフロー向けの**複合指示**処理の改善。
-   トークン使用量が少なく、反復ループが高速な、**より簡潔な応答**。
-   より強力な**ツール/エージェントフレームワーク**互換性とコンテキスト管理（Claude Code、Droid/Factory AI、Cline、Kilo Code、Roo Code、BlackBox）。
-   より高品質な**対話と技術文書**の出力。

## MiniMax M2.5 対 MiniMax M2.5 Highspeed

-   **速度:** `MiniMax-M2.5-highspeed` はMiniMaxドキュメントにおける公式の高速ティアです。
-   **コスト:** MiniMaxの価格設定では、入力コストは同じで、出力コストはhighspeedの方が高くなっています。
-   **互換性:** OpenClawは従来の `MiniMax-M2.5-Lightning` 設定も受け付けますが、新しいセットアップでは `MiniMax-M2.5-highspeed` を推奨します。

## セットアップ方法を選択

### MiniMax OAuth (Coding Plan) — 推奨

**最適な用途:** OAuth経由でMiniMax Coding Planを使用したクイックセットアップ、APIキー不要。バンドルされたOAuthプラグインを有効にして認証します:

```bash
openclaw plugins enable minimax-portal-auth  # すでに読み込まれている場合はスキップ。
openclaw gateway restart  # ゲートウェイがすでに実行中の場合は再起動
openclaw onboard --auth-choice minimax-portal
```

エンドポイントの選択を求められます:

-   **Global** - 国際ユーザー (`api.minimax.io`)
-   **CN** - 中国のユーザー (`api.minimaxi.com`)

詳細は [MiniMax OAuth プラグイン README](https://github.com/openclaw/openclaw/tree/main/extensions/minimax-portal-auth) を参照してください。

### MiniMax M2.5 (APIキー)

**最適な用途:** Anthropic互換APIを備えたホスト型MiniMax。CLI経由で設定:

-   `openclaw configure` を実行
-   **Model/auth** を選択
-   **MiniMax M2.5** を選択

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: { defaults: { model: { primary: "minimax/MiniMax-M2.5" } } },
  models: {
    mode: "merge",
    providers: {
      minimax: {
        baseUrl: "https://api.minimax.io/anthropic",
        apiKey: "${MINIMAX_API_KEY}",
        api: "anthropic-messages",
        models: [
          {
            id: "MiniMax-M2.5",
            name: "MiniMax M2.5",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
          {
            id: "MiniMax-M2.5-highspeed",
            name: "MiniMax M2.5 Highspeed",
            reasoning: true,
            input: ["text"],
            cost: { input: 0.3, output: 1.2, cacheRead: 0.03, cacheWrite: 0.12 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

### MiniMax M2.5 をフォールバックとして使用 (例)

**最適な用途:** 最強の最新世代モデルをプライマリとして維持し、MiniMax M2.5にフォールバック。以下の例では、具体的なプライマリとしてOpusを使用しています。お好みの最新世代プライマリモデルに置き換えてください。

```json
{
  env: { MINIMAX_API_KEY: "sk-..." },
  agents: {
    defaults: {
      models: {
        "anthropic/claude-opus-4-6": { alias: "primary" },
        "minimax/MiniMax-M2.5": { alias: "minimax" },
      },
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["minimax/MiniMax-M2.5"],
      },
    },
  },
}
```

### オプション: LM Studio経由のローカル (手動)

**最適な用途:** LM Studioを使用したローカル推論。強力なハードウェア（例: デスクトップ/サーバー）でLM Studioのローカルサーバーを使用するMiniMax M2.5で、強力な結果が得られています。`openclaw.json` を介して手動で設定:

```json
{
  agents: {
    defaults: {
      model: { primary: "lmstudio/minimax-m2.5-gs32" },
      models: { "lmstudio/minimax-m2.5-gs32": { alias: "Minimax" } },
    },
  },
  models: {
    mode: "merge",
    providers: {
      lmstudio: {
        baseUrl: "http://127.0.0.1:1234/v1",
        apiKey: "lmstudio",
        api: "openai-responses",
        models: [
          {
            id: "minimax-m2.5-gs32",
            name: "MiniMax M2.5 GS32",
            reasoning: false,
            input: ["text"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 196608,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
}
```

## openclaw configure 経由での設定

JSONを編集せずにMiniMaxを設定するには、対話型設定ウィザードを使用します:

1.  `openclaw configure` を実行。
2.  **Model/auth** を選択。
3.  **MiniMax M2.5** を選択。
4.  プロンプトが表示されたらデフォルトモデルを選択。

## 設定オプション

-   `models.providers.minimax.baseUrl`: `https://api.minimax.io/anthropic` (Anthropic互換) を推奨。`https://api.minimax.io/v1` はOpenAI互換ペイロード用のオプションです。
-   `models.providers.minimax.api`: `anthropic-messages` を推奨。`openai-completions` はOpenAI互換ペイロード用のオプションです。
-   `models.providers.minimax.apiKey`: MiniMax APIキー (`MINIMAX_API_KEY`)。
-   `models.providers.minimax.models`: `id`、`name`、`reasoning`、`contextWindow`、`maxTokens`、`cost` を定義。
-   `agents.defaults.models`: 許可リストに含めたいモデルのエイリアスを設定。
-   `models.mode`: 組み込みモデルに加えてMiniMaxを追加したい場合は `merge` のままにします。

## 注意事項

-   モデル参照は `minimax/` の形式です。
-   推奨モデルID: `MiniMax-M2.5` および `MiniMax-M2.5-highspeed`。
-   Coding Plan 使用量 API: `https://api.minimaxi.com/v1/api/openplatform/coding_plan/remains` (コーディングプランキーが必要)。
-   正確なコスト追跡が必要な場合は、`models.json` の価格値を更新してください。
-   MiniMax Coding Plan 紹介リンク (10%オフ): [https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link](https://platform.minimax.io/subscribe/coding-plan?code=DbXJTRClnb&source=link)
-   プロバイダールールについては [/concepts/model-providers](../concepts/model-providers.md) を参照してください。
-   モデル切り替えには `openclaw models list` と `openclaw models set minimax/MiniMax-M2.5` を使用します。

## トラブルシューティング

### 「Unknown model: minimax/MiniMax-M2.5」

これは通常、**MiniMaxプロバイダーが設定されていない**ことを意味します（プロバイダーエントリがなく、MiniMax認証プロファイル/環境変数キーが見つかりません）。この検出に関する修正は **2026.1.12** に含まれています（執筆時点では未リリース）。以下のいずれかで修正:

-   **2026.1.12** にアップグレード（またはソース `main` から実行）し、ゲートウェイを再起動。
-   `openclaw configure` を実行して **MiniMax M2.5** を選択。
-   `models.providers.minimax` ブロックを手動で追加。
-   プロバイダーが注入できるように `MINIMAX_API_KEY`（またはMiniMax認証プロファイル）を設定。

モデルIDは**大文字と小文字を区別する**ことを確認してください:

-   `minimax/MiniMax-M2.5`
-   `minimax/MiniMax-M2.5-highspeed`
-   `minimax/MiniMax-M2.5-Lightning` (レガシー)

その後、以下で再確認:

```bash
openclaw models list
```

[GLM Models](./glm.md)[Moonshot AI](./moonshot.md)