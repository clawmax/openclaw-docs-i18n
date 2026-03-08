title: "OpenClaw API 使用コストとキー管理リファレンス"
description: "チャット、メディア、メモリ、Webツール、スキルなど、APIキーを使用してコストが発生するOpenClawの機能について学びます。使用量と課金を管理しましょう。"
keywords: ["api使用量", "apiコスト", "openclaw api", "apiキー管理", "トークンコスト", "プロバイダー課金", "使用量追跡", "apiキー検出"]
---

  技術リファレンス

  
# API使用量とコスト

このドキュメントでは、**APIキーを呼び出す可能性のある機能**と、そのコストがどこに表示されるかをリストアップします。プロバイダーの使用量や有料APIコールを生成する可能性のあるOpenClawの機能に焦点を当てています。

## コストが表示される場所 (チャット + CLI)

**セッションごとのコストスナップショット**

-   `/status` は、現在のセッションのモデル、コンテキスト使用量、および最後の応答トークンを表示します。
-   モデルが **APIキー認証** を使用する場合、`/status` は最後の返信の **推定コスト** も表示します。

**メッセージごとのコストフッター**

-   `/usage full` は、すべての返信に使用量フッターを追加します。これには **推定コスト** も含まれます（APIキーのみ）。
-   `/usage tokens` はトークンのみを表示します。OAuthフローではドルコストは非表示になります。

**CLI使用量ウィンドウ (プロバイダークォータ)**

-   `openclaw status --usage` と `openclaw channels list` は、プロバイダーの **使用量ウィンドウ** （クォータのスナップショット、メッセージごとのコストではありません）を表示します。

詳細と例については、[トークン使用とコスト](./token-use.md)を参照してください。

## キーが検出される仕組み

OpenClawは以下の場所から認証情報を取得できます:

-   **認証プロファイル** (エージェントごと、`auth-profiles.json`に保存)。
-   **環境変数** (例: `OPENAI_API_KEY`, `BRAVE_API_KEY`, `FIRECRAWL_API_KEY`)。
-   **設定** (`models.providers.*.apiKey`, `tools.web.search.*`, `tools.web.fetch.firecrawl.*`, `memorySearch.*`, `talk.apiKey`)。
-   **スキル** (`skills.entries..apiKey`) - キーをスキルプロセスの環境変数にエクスポートする場合があります。

## キーを使用する可能性のある機能

### 1) コアモデル応答 (チャット + ツール)

すべての返信やツール呼び出しは、**現在のモデルプロバイダー** (OpenAI、Anthropicなど) を使用します。これが使用量とコストの主な発生源です。価格設定については[モデル](../providers/models.md)を、表示については[トークン使用とコスト](./token-use.md)を参照してください。

### 2) メディア理解 (音声/画像/動画)

受信したメディアは、返信が実行される前に要約/書き起こしされることがあります。これにはモデル/プロバイダーAPIが使用されます。

-   音声: OpenAI / Groq / Deepgram (現在はキーが存在する場合に**自動有効化**)。
-   画像: OpenAI / Anthropic / Google。
-   動画: Google。

[メディア理解](../nodes/media-understanding.md)を参照してください。

### 3) メモリ埋め込み + セマンティック検索

セマンティックメモリ検索は、リモートプロバイダー用に設定されている場合に**埋め込みAPI**を使用します:

-   `memorySearch.provider = "openai"` → OpenAI埋め込み
-   `memorySearch.provider = "gemini"` → Gemini埋め込み
-   `memorySearch.provider = "voyage"` → Voyage埋め込み
-   `memorySearch.provider = "mistral"` → Mistral埋め込み
-   `memorySearch.provider = "ollama"` → Ollama埋め込み (ローカル/セルフホスト; 通常はホスト型APIの課金なし)
-   ローカル埋め込みが失敗した場合のリモートプロバイダーへのオプションフォールバック

`memorySearch.provider = "local"` と設定することでローカルに保つことができます（API使用量なし）。[メモリ](../concepts/memory.md)を参照してください。

### 4) Web検索ツール

`web_search` はAPIキーを使用し、プロバイダーに応じて使用料金が発生する可能性があります:

-   **Perplexity Search API**: `PERPLEXITY_API_KEY`
-   **Brave Search API**: `BRAVE_API_KEY` または `tools.web.search.apiKey`
-   **Gemini (Google Search)**: `GEMINI_API_KEY`
-   **Grok (xAI)**: `XAI_API_KEY`
-   **Kimi (Moonshot)**: `KIMI_API_KEY` または `MOONSHOT_API_KEY`

[Webツール](../tools/web.md)を参照してください。

### 5) Webフェッチツール (Firecrawl)

`web_fetch` は、APIキーが存在する場合に **Firecrawl** を呼び出すことができます:

-   `FIRECRAWL_API_KEY` または `tools.web.fetch.firecrawl.apiKey`

Firecrawlが設定されていない場合、このツールは直接フェッチ + readability にフォールバックします（有料APIなし）。[Webツール](../tools/web.md)を参照してください。

### 6) プロバイダー使用量スナップショット (ステータス/ヘルス)

一部のステータスコマンドは、クォータウィンドウや認証ヘルスを表示するために**プロバイダーの使用量エンドポイント**を呼び出します。これらは通常は低ボリュームの呼び出しですが、プロバイダーAPIにアクセスします:

-   `openclaw status --usage`
-   `openclaw models status --json`

[モデルCLI](../cli/models.md)を参照してください。

### 7) 圧縮セーフガード要約

圧縮セーフガードは、セッション履歴を**現在のモデル**を使用して要約することができます。これが実行されるとプロバイダーAPIが呼び出されます。[セッション管理と圧縮](./session-management-compaction.md)を参照してください。

### 8) モデルスキャン / プローブ

`openclaw models scan` はOpenRouterモデルをプローブでき、プローブが有効な場合に `OPENROUTER_API_KEY` を使用します。[モデルCLI](../cli/models.md)を参照してください。

### 9) トーク (音声合成)

トークモードは、設定されている場合に **ElevenLabs** を呼び出すことができます:

-   `ELEVENLABS_API_KEY` または `talk.apiKey`

[トークモード](../nodes/talk.md)を参照してください。

### 10) スキル (サードパーティAPI)

スキルは `skills.entries..apiKey` に `apiKey` を保存できます。スキルがそのキーを外部APIに使用する場合、スキルのプロバイダーに応じてコストが発生する可能性があります。[スキル](../tools/skills.md)を参照してください。

[プロンプトキャッシング](./prompt-caching.md)[トランスクリプトの整理](./transcript-hygiene.md)