

  環境とデバッグ

  
# テスト

OpenClaw には3つの Vitest スイート（ユニット/統合、e2e、ライブ）と、いくつかの Docker ランナーがあります。このドキュメントは「私たちのテスト方法」ガイドです：

-   各スイートがカバーする範囲（および意図的にカバー*しない*範囲）
-   一般的なワークフロー（ローカル、プッシュ前、デバッグ）で実行するコマンド
-   ライブテストが認証情報を発見し、モデル/プロバイダーを選択する方法
-   実世界のモデル/プロバイダー問題に対する回帰テストの追加方法

## クイックスタート

ほとんどの日：

-   完全ゲート（プッシュ前に期待される）：`pnpm build && pnpm check && pnpm test`

テストに触れる場合や追加の確信が必要な場合：

-   カバレッジゲート：`pnpm test:coverage`
-   E2E スイート：`pnpm test:e2e`

実際のプロバイダー/モデルのデバッグ（実際の認証情報が必要）：

-   ライブスイート（モデル + ゲートウェイ ツール/イメージプローブ）：`pnpm test:live`

ヒント：1つの失敗ケースのみが必要な場合は、以下で説明する許可リスト環境変数を使用してライブテストを絞り込むことを推奨します。

## テストスイート（どこで何が実行されるか）

スイートは「現実性の増加」（および不安定性/コストの増加）として考えてください：

### ユニット / 統合（デフォルト）

-   コマンド：`pnpm test`
-   設定：`scripts/test-parallel.mjs` (`vitest.unit.config.ts`, `vitest.extensions.config.ts`, `vitest.gateway.config.ts` を実行)
-   ファイル：`src/**/*.test.ts`, `extensions/**/*.test.ts`
-   範囲：
    -   純粋なユニットテスト
    -   プロセス内統合テスト（ゲートウェイ認証、ルーティング、ツーリング、パース、設定）
    -   既知のバグに対する決定論的回帰テスト
-   期待：
    -   CI で実行される
    -   実際のキーは不要
    -   高速で安定しているべき
-   プールに関する注意：
    -   OpenClaw は Node 22/23 でより高速なユニットシャードのために Vitest `vmForks` を使用します。
    -   Node 24+ では、OpenClaw は Node VM リンクエラー (`ERR_VM_MODULE_LINK_FAILURE` / `module is already linked`) を避けるために自動的に通常の `forks` にフォールバックします。
    -   `OPENCLAW_TEST_VM_FORKS=0` (強制的に `forks`) または `OPENCLAW_TEST_VM_FORKS=1` (強制的に `vmForks`) で手動で上書きできます。

### E2E（ゲートウェイ スモーク）

-   コマンド：`pnpm test:e2e`
-   設定：`vitest.e2e.config.ts`
-   ファイル：`src/**/*.e2e.test.ts`
-   ランタイムデフォルト：
    -   より高速なファイル起動のために Vitest `vmForks` を使用。
    -   適応型ワーカーを使用 (CI: 2-4, ローカル: 4-8)。
    -   デフォルトでサイレントモードで実行し、コンソール I/O オーバーヘッドを削減。
-   便利な上書き：
    -   `OPENCLAW_E2E_WORKERS=` でワーカー数を強制 (最大16)。
    -   `OPENCLAW_E2E_VERBOSE=1` で詳細なコンソール出力を再有効化。
-   範囲：
    -   マルチインスタンスゲートウェイのエンドツーエンド動作
    -   WebSocket/HTTP サーフェス、ノードペアリング、より重いネットワーキング
-   期待：
    -   CI で実行される（パイプラインで有効な場合）
    -   実際のキーは不要
    -   ユニットテストよりも多くの可動部品（より遅くなる可能性あり）

### ライブ（実際のプロバイダー + 実際のモデル）

-   コマンド：`pnpm test:live`
-   設定：`vitest.live.config.ts`
-   ファイル：`src/**/*.live.test.ts`
-   デフォルト：**有効** (`pnpm test:live` によって `OPENCLAW_LIVE_TEST=1` が設定される)
-   範囲：
    -   「このプロバイダー/モデルは実際の認証情報で*今日*実際に動作するか？」
    -   プロバイダーフォーマットの変更、ツール呼び出しの癖、認証問題、レート制限動作の検出
-   期待：
    -   設計上 CI 安定ではない（実際のネットワーク、実際のプロバイダーポリシー、クォータ、障害）
    -   コストがかかる / レート制限を使用する
    -   「すべて」ではなく絞り込んだサブセットの実行を推奨
    -   ライブ実行は不足している API キーを取得するために `~/.profile` をソースする
-   API キーローテーション（プロバイダー固有）：カンマ/セミコロン形式の `*_API_KEYS` または `*_API_KEY_1`, `*_API_KEY_2` を設定（例：`OPENAI_API_KEYS`, `ANTHROPIC_API_KEYS`, `GEMINI_API_KEYS`）または `OPENCLAW_LIVE_*_KEY` によるライブ単位の上書き；テストはレート制限応答でリトライします。

## どのスイートを実行すべきですか？

この決定表を使用してください：

-   ロジック/テストの編集：`pnpm test` を実行（多く変更した場合は `pnpm test:coverage` も）
-   ゲートウェイ ネットワーキング / WS プロトコル / ペアリングに触れる：`pnpm test:e2e` を追加
-   「ボットがダウンしている」/ プロバイダー固有の失敗 / ツール呼び出しのデバッグ：絞り込んだ `pnpm test:live` を実行

## ライブ: Android ノード キャパビリティ スイープ

-   テスト：`src/gateway/android-node.capabilities.live.test.ts`
-   スクリプト：`pnpm android:test:integration`
-   目標：接続された Android ノードによって現在アドバタイズされている**すべてのコマンド**を呼び出し、コマンド契約の動作をアサート。
-   範囲：
    -   前提条件設定/手動セットアップ（スイートはアプリのインストール/実行/ペアリングを行わない）。
    -   選択された Android ノードに対するコマンドごとのゲートウェイ `node.invoke` 検証。
-   必要な事前セットアップ：
    -   Android アプリがすでにゲートウェイに接続 + ペアリング済み。
    -   アプリをフォアグラウンドに維持。
    -   通過を期待するキャパビリティに対して権限/キャプチャ同意が付与済み。
-   オプションのターゲット上書き：
    -   `OPENCLAW_ANDROID_NODE_ID` または `OPENCLAW_ANDROID_NODE_NAME`。
    -   `OPENCLAW_ANDROID_GATEWAY_URL` / `OPENCLAW_ANDROID_GATEWAY_TOKEN` / `OPENCLAW_ANDROID_GATEWAY_PASSWORD`。
-   完全な Android セットアップ詳細：[Android アプリ](../platforms/android.md)

## ライブ: モデル スモーク（プロファイルキー）

ライブテストは、失敗を分離できるように2つのレイヤーに分割されています：

-   「直接モデル」は、プロバイダー/モデルが与えられたキーで少なくとも応答できることを示します。
-   「ゲートウェイ スモーク」は、そのモデルに対して完全なゲートウェイ+エージェントパイプラインが機能することを示します（セッション、履歴、ツール、サンドボックスポリシーなど）。

### レイヤー 1: 直接モデル完了（ゲートウェイなし）

-   テスト：`src/agents/models.profiles.live.test.ts`
-   目標：
    -   発見されたモデルを列挙
    -   `getApiKeyForModel` を使用して認証情報を持つモデルを選択
    -   モデルごとに小さな完了を実行（必要に応じて対象を絞った回帰テストも）
-   有効化方法：
    -   `pnpm test:live` (または Vitest を直接呼び出す場合は `OPENCLAW_LIVE_TEST=1`)
-   `OPENCLAW_LIVE_MODELS=modern` (または `all`、modern のエイリアス) を設定して実際にこのスイートを実行；それ以外の場合は `pnpm test:live` をゲートウェイ スモークに集中させるためにスキップ
-   モデル選択方法：
    -   `OPENCLAW_LIVE_MODELS=modern` でモダン許可リストを実行 (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_MODELS=all` はモダン許可リストのエイリアス
    -   または `OPENCLAW_LIVE_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,..."` (カンマ許可リスト)
-   プロバイダー選択方法：
    -   `OPENCLAW_LIVE_PROVIDERS="google,google-antigravity,google-gemini-cli"` (カンマ許可リスト)
-   キーの取得元：
    -   デフォルト：プロファイルストアと環境変数のフォールバック
    -   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` を設定して**プロファイルストア**のみを強制
-   これが存在する理由：
    -   「プロバイダー API が壊れている / キーが無効」と「ゲートウェイ エージェントパイプラインが壊れている」を分離
    -   小さく分離された回帰テストを含む（例：OpenAI Responses/Codex Responses 推論再生 + ツール呼び出しフロー）

### レイヤー 2: ゲートウェイ + 開発エージェント スモーク（「@openclaw」が実際に行うこと）

-   テスト：`src/gateway/gateway-models.profiles.live.test.ts`
-   目標：
    -   プロセス内ゲートウェイを起動
    -   `agent:dev:*` セッションを作成/パッチ（実行ごとにモデル上書き）
    -   キーを持つモデルを反復処理し、以下をアサート：
        -   「意味のある」応答（ツールなし）
        -   実際のツール呼び出しが機能する（読み取りプローブ）
        -   オプションの追加ツールプローブ（実行+読み取りプローブ）
        -   OpenAI 回帰パス（ツール呼び出しのみ → フォローアップ）が機能し続ける
-   プローブ詳細（失敗を迅速に説明できるように）：
    -   `read` プローブ：テストはワークスペースにノンスファイルを書き込み、エージェントにそれを`read`してノンスをエコーバックするよう要求。
    -   `exec+read` プローブ：テストはエージェントにノンスを一時ファイルに`exec`-書き込みし、それを`read`して戻すよう要求。
    -   image プローブ：テストは生成された PNG（猫 + ランダム化されたコード）を添付し、モデルが `cat <コード>` を返すことを期待。
    -   実装リファレンス：`src/gateway/gateway-models.profiles.live.test.ts` および `src/gateway/live-image-probe.ts`。
-   有効化方法：
    -   `pnpm test:live` (または Vitest を直接呼び出す場合は `OPENCLAW_LIVE_TEST=1`)
-   モデル選択方法：
    -   デフォルト：モダン許可リスト (Opus/Sonnet/Haiku 4.5, GPT-5.x + Codex, Gemini 3, GLM 4.7, MiniMax M2.5, Grok 4)
    -   `OPENCLAW_LIVE_GATEWAY_MODELS=all` はモダン許可リストのエイリアス
    -   または `OPENCLAW_LIVE_GATEWAY_MODELS="provider/model"` (またはカンマリスト) を設定して絞り込み
-   プロバイダー選択方法（「OpenRouter すべて」を避ける）：
    -   `OPENCLAW_LIVE_GATEWAY_PROVIDERS="google,google-antigravity,google-gemini-cli,openai,anthropic,zai,minimax"` (カンマ許可リスト)
-   ツール + イメージプローブはこのライブテストで常にオン：
    -   `read` プローブ + `exec+read` プローブ（ツールストレス）
    -   モデルがイメージ入力サポートをアドバタイズする場合にイメージプローブを実行
    -   フロー（高レベル）：
        -   テストは「CAT」+ ランダムコード (`src/gateway/live-image-probe.ts`) を含む小さな PNG を生成
        -   `agent` `attachments: [{ mimeType: "image/png", content: "" }]` 経由で送信
        -   ゲートウェイは添付ファイルを `images[]` にパース (`src/gateway/server-methods/agent.ts` + `src/gateway/chat-attachments.ts`)
        -   組み込みエージェントはマルチモーダルユーザーメッセージをモデルに転送
        -   アサーション：応答に `cat` + コードが含まれる（OCR 許容：軽微な間違いは許可）

ヒント：マシンでテストできるもの（および正確な `provider/model` ID）を確認するには、以下を実行：

```bash
openclaw models list
openclaw models list --json
```

## ライブ: Anthropic セットアップトークン スモーク

-   テスト：`src/agents/anthropic.setup-token.live.test.ts`
-   目標：Claude Code CLI セットアップトークン（または貼り付けられたセットアップトークンプロファイル）が Anthropic プロンプトを完了できることを検証。
-   有効化：
    -   `pnpm test:live` (または Vitest を直接呼び出す場合は `OPENCLAW_LIVE_TEST=1`)
    -   `OPENCLAW_LIVE_SETUP_TOKEN=1`
-   トークンソース（いずれかを選択）：
    -   プロファイル：`OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test`
    -   生トークン：`OPENCLAW_LIVE_SETUP_TOKEN_VALUE=sk-ant-oat01-...`
-   モデル上書き（オプション）：
    -   `OPENCLAW_LIVE_SETUP_TOKEN_MODEL=anthropic/claude-opus-4-6`

セットアップ例：

```bash
openclaw models auth paste-token --provider anthropic --profile-id anthropic:setup-token-test
OPENCLAW_LIVE_SETUP_TOKEN=1 OPENCLAW_LIVE_SETUP_TOKEN_PROFILE=anthropic:setup-token-test pnpm test:live src/agents/anthropic.setup-token.live.test.ts
```

## ライブ: CLI バックエンド スモーク（Claude Code CLI またはその他のローカル CLI）

-   テスト：`src/gateway/gateway-cli-backend.live.test.ts`
-   目標：デフォルト設定に触れずに、ローカル CLI バックエンドを使用してゲートウェイ + エージェントパイプラインを検証。
-   有効化：
    -   `pnpm test:live` (または Vitest を直接呼び出す場合は `OPENCLAW_LIVE_TEST=1`)
    -   `OPENCLAW_LIVE_CLI_BACKEND=1`
-   デフォルト：
    -   モデル：`claude-cli/claude-sonnet-4-6`
    -   コマンド：`claude`
    -   引数：`["-p","--output-format","json","--permission-mode","bypassPermissions"]`
-   上書き（オプション）：
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-opus-4-6"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_MODEL="codex-cli/gpt-5.4"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_COMMAND="/full/path/to/claude"`
    -   `OPENCLAW_LIVE_CLI_BACKEND_ARGS='["-p","--output-format","json","--permission-mode","bypassPermissions"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_CLEAR_ENV='["ANTHROPIC_API_KEY","ANTHROPIC_API_KEY_OLD"]'`
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_PROBE=1` で実際のイメージ添付ファイルを送信（パスはプロンプトに注入される）。
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_ARG="--image"` でイメージファイルパスをプロンプト注入ではなく CLI 引数として渡す。
    -   `OPENCLAW_LIVE_CLI_BACKEND_IMAGE_MODE="repeat"` (または `"list"`) で `IMAGE_ARG` 設定時のイメージ引数の渡し方を制御。
    -   `OPENCLAW_LIVE_CLI_BACKEND_RESUME_PROBE=1` で2つ目のターンを送信し、再開フローを検証。
-   `OPENCLAW_LIVE_CLI_BACKEND_DISABLE_MCP_CONFIG=0` で Claude Code CLI MCP 設定を有効に維持（デフォルトは一時的な空ファイルで MCP 設定を無効化）。

例：

```
OPENCLAW_LIVE_CLI_BACKEND=1 \
  OPENCLAW_LIVE_CLI_BACKEND_MODEL="claude-cli/claude-sonnet-4-6" \
  pnpm test:live src/gateway/gateway-cli-backend.live.test.ts
```

### 推奨ライブレシピ

絞り込んだ明示的な許可リストが最速で最も不安定さが少ない：

-   単一モデル、直接（ゲートウェイなし）：
    -   `OPENCLAW_LIVE_MODELS="openai/gpt-5.2" pnpm test:live src/agents/models.profiles.live.test.ts`
-   単一モデル、ゲートウェイ スモーク：
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   複数プロバイダーにわたるツール呼び出し：
    -   `OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,anthropic/claude-opus-4-6,google/gemini-3-flash-preview,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
-   Google フォーカス（Gemini API キー + Antigravity）：
    -   Gemini (API キー)：`OPENCLAW_LIVE_GATEWAY_MODELS="google/gemini-3-flash-preview" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`
    -   Antigravity (OAuth)：`OPENCLAW_LIVE_GATEWAY_MODELS="google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-pro-high" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

注意：

-   `google/...` は Gemini API (API キー) を使用。
-   `google-antigravity/...` は Antigravity OAuth ブリッジ (Cloud Code Assist スタイルのエージェントエンドポイント) を使用。
-   `google-gemini-cli/...` はマシン上のローカル Gemini CLI を使用（別の認証 + ツーリングの癖）。
-   Gemini API 対 Gemini CLI：
    -   API：OpenClaw は Google のホストされた Gemini API を HTTP 経由で呼び出す（API キー / プロファイル認証）；これがほとんどのユーザーが「Gemini」と言うときに意味するもの。
    -   CLI：OpenClaw はローカル `gemini` バイナリをシェルアウト；独自の認証を持ち、異なる動作をする可能性がある（ストリーミング/ツールサポート/バージョンスキュー）。

## ライブ: モデル マトリックス（カバーする内容）

固定の「CI モデルリスト」はありません（ライブはオプトイン）が、これらはキーを持つ開発マシンで定期的にカバーする**推奨**モデルです。

### モダン スモークセット（ツール呼び出し + イメージ）

これが「一般的なモデル」実行で、動作し続けることが期待されます：

-   OpenAI (非 Codex)：`openai/gpt-5.2` (オプション：`openai/gpt-5.1`)
-   OpenAI Codex：`openai-codex/gpt-5.4`
-   Anthropic：`anthropic/claude-opus-4-6` (または `anthropic/claude-sonnet-4-5`)
-   Google (Gemini API)：`google/gemini-3-pro-preview` および `google/gemini-3-flash-preview` (古い Gemini 2.x モデルは避ける)
-   Google (Antigravity)：`google-antigravity/claude-opus-4-6-thinking` および `google-antigravity/gemini-3-flash`
-   Z.AI (GLM)：`zai/glm-4.7`
-   MiniMax：`minimax/minimax-m2.5`

ツール + イメージでゲートウェイ スモークを実行：`OPENCLAW_LIVE_GATEWAY_MODELS="openai/gpt-5.2,openai-codex/gpt-5.4,anthropic/claude-opus-4-6,google/gemini-3-pro-preview,google/gemini-3-flash-preview,google-antigravity/claude-opus-4-6-thinking,google-antigravity/gemini-3-flash,zai/glm-4.7,minimax/minimax-m2.5" pnpm test:live src/gateway/gateway-models.profiles.live.test.ts`

### ベースライン: ツール呼び出し（読み取り + オプション実行）

プロバイダーファミリーごとに少なくとも1つ選択：

-   OpenAI：`openai/gpt-5.2` (または `openai/gpt-5-mini`)
-   Anthropic：`anthropic/claude-opus-4-6` (または `anthropic/claude-sonnet-4-5`)
-   Google：`google/gemini-3-flash-preview` (または `google/gemini-3-pro-preview`)
-   Z.AI (GLM)：`zai/glm-4.7`
-   MiniMax：`minimax/minimax-m2.5`

オプションの追加カバレッジ（あると良い）：

-   xAI：`xai/grok-4` (または最新の利用可能なもの)
-   Mistral：`mistral/`… (「ツール」対応モデルから1つ選択、有効なもの)
-   Cerebras：`cerebras/`… (アクセス権がある場合)
-   LM Studio：`lmstudio/`… (ローカル；ツール呼び出しは API モードに依存)

### ビジョン: イメージ送信（添付ファイル → マルチモーダルメッセージ）

イメージプローブを実行するために、`OPENCLAW_LIVE_GATEWAY_MODELS` に少なくとも1つのイメージ対応モデルを含める（Claude/Gemini/OpenAI ビジョン対応バリアントなど）。

### アグリゲーター / 代替ゲートウェイ

キーが有効な場合、以下経由のテストもサポートします：

-   OpenRouter：`openrouter/...` (数百のモデル；`openclaw models scan` を使用してツール+イメージ対応候補を検索)
-   OpenCode Zen：`opencode/...` (`OPENCODE_API_KEY` / `OPENCODE_ZEN_API_KEY` 経由の認証)

ライブマトリックスに含めることができるその他のプロバイダー（認証情報/設定がある場合）：

-   組み込み：`openai`, `openai-codex`, `anthropic`, `google`, `google-vertex`, `google-antigravity`, `google-gemini-cli`, `zai`, `openrouter`, `opencode`, `xai`, `groq`, `cerebras`, `mistral`, `github-copilot`
-   `models.providers` 経由（カスタムエンドポイント）：`minimax` (クラウド/API)、および任意の OpenAI/Anthropic 互換プロキシ (LM Studio, vLLM, LiteLLM など)

ヒント：ドキュメントに「すべてのモデル」をハードコードしようとしないでください。権威あるリストは、マシン上で `discoverModels(...)` が返すものと、利用可能なキーです。

## 認証情報（コミットしない）

ライブテストは CLI と同じ方法で認証情報を発見します。実用的な意味：

-   CLI が動作する場合、ライブテストは同じキーを見つけるはず。
-   ライブテストが「認証情報なし」と言う場合、`openclaw models list` / モデル選択のデバッグと同じ方法でデバッグ。
-   プロファイルストア：`~/.openclaw/credentials/` (推奨；テストで「プロファイルキー」が意味するもの)
-   設定：`~/.openclaw/openclaw.json` (または `OPENCLAW_CONFIG_PATH`)

環境変数キーに依存したい場合（例：`~/.profile` でエクスポート）、`source ~/.profile` 後にローカルテストを実行するか、以下の Docker ランナーを使用してください（コンテナに `~/.profile` をマウントできます）。

## Deepgram ライブ（音声文字起こし）

-   テスト：`src/media-understanding/providers/deepgram/audio.live.test.ts`
-   有効化：`DEEPGRAM_API_KEY=... DEEPGRAM_LIVE_TEST=1 pnpm test:live src/media-understanding/providers/deepgram/audio.live.test.ts`

## BytePlus コーディングプラン ライブ

-   テスト：`src/agents/byteplus.live.test.ts`
-   有効化：`BYTEPLUS_API_KEY=... BYTEPLUS_LIVE_TEST=1 pnpm test:live src/agents/byteplus.live.test.ts`
-   オプションモデル上書き：`BYTEPLUS_CODING_MODEL=ark-code-latest`

## Docker ランナー（オプションの「Linux で動作する」チェック）

これらはリポジトリ Docker イメージ内で `pnpm test:live` を実行し、ローカル設定ディレクトリとワークスペースをマウントします（マウントされている場合は `~/.profile` もソース）：

-   直接モデル：`pnpm test:docker:live-models` (スクリプト：`scripts/test-live-models-docker.sh`)
-   ゲートウェイ + 開発エージェント：`pnpm test:docker:live-gateway` (スクリプト：`scripts/test-live-gateway-models-docker.sh`)
-   オンボーディングウィザード (TTY, 完全なスキャフォールディング)：`pnpm test:docker:onboard` (スクリプト：`scripts/e2e/onboard-docker.sh`)
-   ゲートウェイ ネットワーキング (2つのコンテナ, WS 認証 + ヘルス)：`pnpm test:docker:gateway-network` (スクリプト：`scripts/e2e/gateway-network-docker.sh`)
-   プラグイン (カスタム拡張ロード + レジストリスモーク)：`pnpm test:docker:plugins` (スクリプト：`scripts/e2e/plugins-docker.sh`)

手動 ACP 平文スレッド スモーク（CI ではない）：

-   `bun scripts/dev/discord-acp-plain-language-smoke.ts --channel <discord-channel-id> ...`
-   このスクリプトは回帰/デバッグワークフロー用に保持。ACP スレッドルーティング検証に再度必要になる可能性があるため、削除しないでください。

便利な環境変数：

-   `OPENCLAW_CONFIG_DIR=...` (デフォルト：`~/.openclaw`) が `/home/node/.openclaw` にマウント
-   `OPENCLAW_WORKSPACE_DIR=...` (デフォルト：`~/.openclaw/workspace`) が `/home/node/.openclaw/workspace` にマウント
-   `OPENCLAW_PROFILE_FILE=...` (デフォルト：`~/.profile`) が `/home/node/.profile` にマウントされ、テスト実行前にソース
-   `OPENCLAW_LIVE_GATEWAY_MODELS=...` / `OPENCLAW_LIVE_MODELS=...` で実行を絞り込み
-   `OPENCLAW_LIVE_REQUIRE_PROFILE_KEYS=1` で認証情報がプロファイルストアから来ることを保証（環境変数ではない）

## ドキュメント健全性

ドキュメント編集後にドキュメントチェックを実行：`pnpm docs:list`。

## オフライン回帰（CI 安全）

これらは実際のプロバイダーなしの「実際のパイプライン」回帰テストです：

-   ゲートウェイ ツール呼び出し（モック OpenAI, 実際のゲートウェイ + エージェントループ）：`src/gateway/gateway.test.ts` (ケース：「モック OpenAI ツール呼び出しをゲートウェイ エージェントループ経由でエンドツーエンドで実行」)
-   ゲートウェイウィザード (WS `wizard.start`/`wizard.next`, 設定 + 認証強制を書き込み)：`src/gateway/gateway.test.ts` (ケース：「WS 経由でウィザードを実行し、認証トークン設定を書き込む」)

## エージェント信頼性評価（スキル）

すでに「エージェント信頼性評価」のように動作するいくつかの CI 安全テストがあります：

-   実際のゲートウェイ + エージェントループを通じたモックツール呼び出し (`src/gateway/gateway.test.ts`)。
-   セッション配線と設定効果を検証するエンドツーエンドウィザードフロー (`src/gateway/gateway.test.ts`)。

スキル用にまだ不足しているもの（[スキル](../tools/skills.md) 参照）：

-   **意思決定：** スキルがプロンプトにリストされている場合、エージェントは適切なスキルを選択するか（または無関係なものを避けるか）？
-   **コンプライアンス：** エージェントは使用前に `SKILL.md` を読み、必要な手順/引数に従うか？
-   **ワークフロー契約：** ツール順序、セッション履歴の引き継ぎ、サンドボックス境界をアサートするマルチターンシナリオ。

将来の評価はまず決定論的に保つべき：

-   モックプロバイダーを使用してツール呼び出し + 順序、スキルファイル読み取り、セッション配線をアサートするシナリオランナー。
-   スキル中心のシナリオの小さなスイート（使用 vs 回避、ゲーティング、プロンプトインジェクション）。
-   CI 安全スイートが整った後のみのオプションライブ評価（オプトイン、環境変数ゲート）。

## 回帰