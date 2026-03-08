title: "OpenClaw AI メディア理解ノード設定ガイド"
description: "OpenClaw AI で画像、音声、動画を要約する設定方法を学びます。最適なメディア処理のために、プロバイダーAPI、CLIフォールバック、添付ファイルポリシーを設定します。"
keywords: ["メディア理解", "openclaw ai", "音声文字起こし", "画像要約", "動画分析", "ai設定", "マルチモーダルai", "メディア処理"]
---

  メディアとデバイス

  
# メディア理解

OpenClawは、返信パイプラインが実行される前に、受信したメディア（画像/音声/動画）を**要約**できます。ローカルツールやプロバイダーキーが利用可能な場合を自動検出し、無効化やカスタマイズも可能です。理解機能がオフの場合でも、モデルは通常通り元のファイル/URLを受け取ります。

## 目的

-   オプション: 受信メディアを短いテキストに事前要約し、ルーティングの高速化とコマンド解析の向上を図る。
-   元のメディアをモデルに確実に配信する（常に）。
-   **プロバイダーAPI**と**CLIフォールバック**をサポートする。
-   複数のモデルを順序付きフォールバック（エラー/サイズ/タイムアウト）で利用可能にする。

## 高レベルの動作

1.  受信した添付ファイルを収集する（`MediaPaths`、`MediaUrls`、`MediaTypes`）。
2.  有効化された各機能（画像/音声/動画）について、ポリシー（デフォルト: **最初の**ファイル）に従って添付ファイルを選択する。
3.  最初に適格なモデルエントリを選択する（サイズ + 機能 + 認証）。
4.  モデルが失敗した場合、またはメディアが大きすぎる場合、**次のエントリにフォールバック**する。
5.  成功時:
    -   `Body` は `[Image]`、`[Audio]`、または `[Video]` ブロックになる。
    -   音声は `{{Transcript}}` を設定する。コマンド解析では、キャプションテキストが存在する場合はそれを使用し、なければ文字起こしを使用する。
    -   キャプションはブロック内の `User text:` として保持される。

理解が失敗した場合、または無効化されている場合、**返信フローは**元の本文と添付ファイルで**続行される**。

## 設定概要

`tools.media` は、**共有モデル**と機能ごとのオーバーライドをサポートします:

-   `tools.media.models`: 共有モデルリスト（`capabilities` で制限）。
-   `tools.media.image` / `tools.media.audio` / `tools.media.video`:
    -   デフォルト値（`prompt`、`maxChars`、`maxBytes`、`timeoutSeconds`、`language`）
    -   プロバイダーオーバーライド（`baseUrl`、`headers`、`providerOptions`）
    -   Deepgram音声オプション（`tools.media.audio.providerOptions.deepgram` 経由）
    -   音声文字起こしエコー制御（`echoTranscript`、デフォルト `false`； `echoFormat`）
    -   オプションの**機能ごとの `models` リスト**（共有モデルより優先）
    -   `attachments` ポリシー（`mode`、`maxAttachments`、`prefer`）
    -   `scope`（チャンネル/チャットタイプ/セッションキーによるオプションの制限）
-   `tools.media.concurrency`: 同時実行可能な機能実行の最大数（デフォルト **2**）。

```json
{
  tools: {
    media: {
      models: [
        /* shared list */
      ],
      image: {
        /* optional overrides */
      },
      audio: {
        /* optional overrides */
        echoTranscript: true,
        echoFormat: '📝 "{transcript}"',
      },
      video: {
        /* optional overrides */
      },
    },
  },
}
```

### モデルエントリ

各 `models[]` エントリは、**プロバイダー**または**CLI**のいずれかです:

```json
{
  type: "provider", // 省略時はデフォルト
  provider: "openai",
  model: "gpt-5.2",
  prompt: "Describe the image in <= 500 chars.",
  maxChars: 500,
  maxBytes: 10485760,
  timeoutSeconds: 60,
  capabilities: ["image"], // オプション、マルチモーダルエントリで使用
  profile: "vision-profile",
  preferredProfile: "vision-fallback",
}
```

```json
{
  type: "cli",
  command: "gemini",
  args: [
    "-m",
    "gemini-3-flash",
    "--allowed-tools",
    "read_file",
    "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
  ],
  maxChars: 500,
  maxBytes: 52428800,
  timeoutSeconds: 120,
  capabilities: ["video", "image"],
}
```

CLIテンプレートでは以下も使用できます:

-   `{{MediaDir}}`（メディアファイルを含むディレクトリ）
-   `{{OutputDir}}`（この実行のために作成されたスクラッチディレクトリ）
-   `{{OutputBase}}`（スクラッチファイルのベースパス、拡張子なし）

## デフォルトと制限

推奨デフォルト:

-   `maxChars`: 画像/動画の場合 **500**（短く、コマンドに適した長さ）
-   `maxChars`: 音声の場合 **未設定**（制限を設定しない限り完全な文字起こし）
-   `maxBytes`:
    -   画像: **10MB**
    -   音声: **20MB**
    -   動画: **50MB**

ルール:

-   メディアが `maxBytes` を超える場合、そのモデルはスキップされ、**次のモデルが試行される**。
-   **1024バイト**未満の音声ファイルは、空/破損として扱われ、プロバイダー/CLI文字起こしの前にスキップされる。
-   モデルが `maxChars` より多くの文字を返した場合、出力は切り詰められる。
-   `prompt` のデフォルトは、シンプルな「Describe the .」に `maxChars` のガイダンスを加えたもの（画像/動画のみ）。
-   `.enabled: true` が設定されていてもモデルが設定されていない場合、OpenClawはその機能をサポートするプロバイダーの**アクティブな返信モデル**を試行する。

### メディア理解の自動検出（デフォルト）

`tools.media..enabled` が `false` に設定されておらず、モデルが設定されていない場合、OpenClawは以下の順序で自動検出を行い、**最初に動作するオプションで停止**します:

1.  **ローカルCLI**（音声のみ；インストール済みの場合）
    -   `sherpa-onnx-offline`（エンコーダー/デコーダー/ジョイナー/トークンを含む `SHERPA_ONNX_MODEL_DIR` が必要）
    -   `whisper-cli`（`whisper-cpp`；`WHISPER_CPP_MODEL` またはバンドルされたtinyモデルを使用）
    -   `whisper`（Python CLI；モデルを自動ダウンロード）
2.  **Gemini CLI**（`gemini`）、`read_many_files` を使用
3.  **プロバイダーキー**
    -   音声: OpenAI → Groq → Deepgram → Google
    -   画像: OpenAI → Anthropic → Google → MiniMax
    -   動画: Google

自動検出を無効にするには、以下を設定します:

```json
{
  tools: {
    media: {
      audio: {
        enabled: false,
      },
    },
  },
}
```

注意: バイナリ検出はmacOS/Linux/Windows間でベストエフォートです。CLIが `PATH` 上にあること（`~` は展開されます）、または完全なコマンドパスで明示的なCLIモデルを設定してください。

### プロキシ環境サポート（プロバイダーモデル）

プロバイダーベースの**音声**および**動画**メディア理解が有効な場合、OpenClawはプロバイダーHTTPコールに対して標準的なアウトバウンドプロキシ環境変数を尊重します:

-   `HTTPS_PROXY`
-   `HTTP_PROXY`
-   `https_proxy`
-   `http_proxy`

プロキシ環境変数が設定されていない場合、メディア理解は直接エグレスを使用します。プロキシ値が不正な形式の場合、OpenClawは警告をログ出力し、直接フェッチにフォールバックします。

## 機能（オプション）

`capabilities` を設定した場合、そのエントリはそれらのメディアタイプに対してのみ実行されます。共有リストの場合、OpenClawはデフォルトを推測できます:

-   `openai`、`anthropic`、`minimax`: **画像**
-   `google`（Gemini API）: **画像 + 音声 + 動画**
-   `groq`: **音声**
-   `deepgram`: **音声**

CLIエントリの場合、予期しないマッチを避けるために**明示的に `capabilities` を設定**してください。`capabilities` を省略した場合、エントリはそれが含まれるリストに対して適格となります。

## プロバイダーサポートマトリックス（OpenClaw統合）

| 機能 | プロバイダー統合 | 備考 |
| --- | --- | --- |
| 画像 | OpenAI / Anthropic / Google / `pi-ai`経由のその他 | レジストリ内の画像対応モデルはすべて動作します。 |
| 音声 | OpenAI, Groq, Deepgram, Google, Mistral | プロバイダー文字起こし（Whisper/Deepgram/Gemini/Voxtral）。 |
| 動画 | Google（Gemini API） | プロバイダー動画理解。 |

## モデル選択ガイダンス

-   品質と安全性が重要な場合、各メディア機能に対して利用可能な最新世代の最強モデルを優先する。
-   信頼できない入力を扱うツール対応エージェントの場合、古い/弱いメディアモデルは避ける。
-   可用性のために、機能ごとに少なくとも1つのフォールバックを用意する（高品質モデル + 高速/低コストモデル）。
-   CLIフォールバック（`whisper-cli`、`whisper`、`gemini`）は、プロバイダーAPIが利用できない場合に有用。
-   `parakeet-mlx` 注意: `--output-dir` を使用する場合、OpenClawは出力形式が `txt`（または未指定）の場合、`<output-dir>/<media-basename>.txt` を読み取る。`txt` 以外の形式はstdoutにフォールバックする。

## 添付ファイルポリシー

機能ごとの `attachments` は、どの添付ファイルを処理するかを制御します:

-   `mode`: `first`（デフォルト）または `all`
-   `maxAttachments`: 処理する数の上限（デフォルト **1**）
-   `prefer`: `first`、`last`、`path`、`url`

`mode: "all"` の場合、出力は `[Image 1/2]`、`[Audio 2/2]` などとラベル付けされます。

## 設定例

### 1) 共有モデルリスト + オーバーライド

```json
{
  tools: {
    media: {
      models: [
        { provider: "openai", model: "gpt-5.2", capabilities: ["image"] },
        {
          provider: "google",
          model: "gemini-3-flash-preview",
          capabilities: ["image", "audio", "video"],
        },
        {
          type: "cli",
          command: "gemini",
          args: [
            "-m",
            "gemini-3-flash",
            "--allowed-tools",
            "read_file",
            "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
          ],
          capabilities: ["image", "video"],
        },
      ],
      audio: {
        attachments: { mode: "all", maxAttachments: 2 },
      },
      video: {
        maxChars: 500,
      },
    },
  },
}
```

### 2) 音声 + 動画のみ（画像オフ）

```json
{
  tools: {
    media: {
      audio: {
        enabled: true,
        models: [
          { provider: "openai", model: "gpt-4o-mini-transcribe" },
          {
            type: "cli",
            command: "whisper",
            args: ["--model", "base", "{{MediaPath}}"],
          },
        ],
      },
      video: {
        enabled: true,
        maxChars: 500,
        models: [
          { provider: "google", model: "gemini-3-flash-preview" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 3) オプションの画像理解

```json
{
  tools: {
    media: {
      image: {
        enabled: true,
        maxBytes: 10485760,
        maxChars: 500,
        models: [
          { provider: "openai", model: "gpt-5.2" },
          { provider: "anthropic", model: "claude-opus-4-6" },
          {
            type: "cli",
            command: "gemini",
            args: [
              "-m",
              "gemini-3-flash",
              "--allowed-tools",
              "read_file",
              "Read the media at {{MediaPath}} and describe it in <= {{MaxChars}} characters.",
            ],
          },
        ],
      },
    },
  },
}
```

### 4) マルチモーダル単一エントリ（明示的な機能）

```json
{
  tools: {
    media: {
      image: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      audio: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
      video: {
        models: [
          {
            provider: "google",
            model: "gemini-3-pro-preview",
            capabilities: ["image", "video", "audio"],
          },
        ],
      },
    },
  },
}
```

## ステータス出力

メディア理解が実行されると、`/status` に短い概要行が含まれます:

```
📎 Media: image ok (openai/gpt-5.2) · audio skipped (maxBytes)
```

これは、機能ごとの結果と、適用可能な場合は選択されたプロバイダー/モデルを示します。

## 注意点

-   理解は**ベストエフォート**です。エラーが発生しても返信はブロックされません。
-   添付ファイルは、理解が無効化されている場合でもモデルに渡されます。
-   `scope` を使用して、理解が実行される場所を制限できます（例: DMのみ）。

## 関連ドキュメント

-   [設定](../gateway/configuration.md)
-   [画像とメディアサポート](./images.md)

[ノードのトラブルシューティング](./troubleshooting.md)[画像とメディアサポート](./images.md)