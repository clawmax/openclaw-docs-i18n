

  CLI コマンド

  
# acp

[Agent Client Protocol (ACP)](https://agentclientprotocol.com/) ブリッジを実行し、OpenClaw Gateway と通信します。このコマンドは IDE 向けに stdio 経由で ACP を話し、プロンプトを WebSocket 経由で Gateway に転送します。ACP セッションを Gateway セッションキーにマッピングして維持します。

## 使用方法

```bash
openclaw acp

# リモート Gateway
openclaw acp --url wss://gateway-host:18789 --token <token>

# リモート Gateway (ファイルからトークン読み込み)
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# 既存のセッションキーにアタッチ
openclaw acp --session agent:main:main

# ラベルでアタッチ (既に存在している必要があります)
openclaw acp --session-label "support inbox"

# 最初のプロンプトの前にセッションキーをリセット
openclaw acp --session agent:main:main --reset-session
```

## ACP クライアント (デバッグ)

組み込みの ACP クライアントを使用して、IDE なしでブリッジを動作確認できます。ACP ブリッジを起動し、対話的にプロンプトを入力できます。

```bash
openclaw acp client

# 起動するブリッジをリモート Gateway に向ける
openclaw acp client --server-args --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token

# サーバーコマンドを上書き (デフォルト: openclaw)
openclaw acp client --server "node" --server-args openclaw.mjs acp --url ws://127.0.0.1:19001
```

権限モデル (クライアントデバッグモード):

-   自動承認は許可リストベースで、信頼されたコアツール ID にのみ適用されます。
-   `read` の自動承認は、現在の作業ディレクトリ (`--cwd` 設定時はそのディレクトリ) にスコープされます。
-   不明/非コアのツール名、スコープ外の読み取り、危険なツールは常に明示的なプロンプト承認が必要です。
-   サーバーから提供される `toolCall.kind` は信頼できないメタデータとして扱われます (認可ソースではありません)。

## 使用方法

IDE (または他のクライアント) が Agent Client Protocol を話し、OpenClaw Gateway セッションを駆動させたい場合に ACP を使用します。

1.  Gateway が実行中であることを確認します (ローカルまたはリモート)。
2.  Gateway ターゲットを設定します (設定またはフラグ)。
3.  IDE が stdio 経由で `openclaw acp` を実行するように設定します。

設定例 (永続化):

```bash
openclaw config set gateway.remote.url wss://gateway-host:18789
openclaw config set gateway.remote.token <token>
```

直接実行例 (設定書き込みなし):

```bash
openclaw acp --url wss://gateway-host:18789 --token <token>
# ローカルプロセスの安全性のため、こちらが推奨されます
openclaw acp --url wss://gateway-host:18789 --token-file ~/.openclaw/gateway.token
```

## エージェントの選択

ACP は直接エージェントを選択しません。Gateway セッションキーによってルーティングされます。特定のエージェントをターゲットにするには、エージェントスコープのセッションキーを使用します:

```bash
openclaw acp --session agent:main:main
openclaw acp --session agent:design:main
openclaw acp --session agent:qa:bug-123
```

各 ACP セッションは単一の Gateway セッションキーにマッピングされます。1 つのエージェントは多くのセッションを持つことができます。キーまたはラベルを上書きしない限り、ACP は隔離された `acp:` セッションをデフォルトとします。

## Zed エディタのセットアップ

`~/.config/zed/settings.json` にカスタム ACP エージェントを追加します (または Zed の設定 UI を使用):

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp"],
      "env": {}
    }
  }
}
```

特定の Gateway またはエージェントをターゲットにするには:

```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": [
        "acp",
        "--url",
        "wss://gateway-host:18789",
        "--token",
        "<token>",
        "--session",
        "agent:design:main"
      ],
      "env": {}
    }
  }
}
```

Zed で、エージェントパネルを開き、「OpenClaw ACP」を選択してスレッドを開始します。

## セッションマッピング

デフォルトでは、ACP セッションは `acp:` プレフィックスを持つ隔離された Gateway セッションキーを取得します。既知のセッションを再利用するには、セッションキーまたはラベルを渡します:

-   `--session `: 特定の Gateway セッションキーを使用します。
-   `--session-label `: 既存のセッションをラベルで解決します。
-   `--reset-session`: そのキーに対して新しいセッション ID を作成します (同じキー、新しいトランスクリプト)。

ACP クライアントがメタデータをサポートしている場合、セッションごとに上書きできます:

```json
{
  "_meta": {
    "sessionKey": "agent:main:main",
    "sessionLabel": "support inbox",
    "resetSession": true
  }
}
```

セッションキーについて詳しくは、[/concepts/session](../concepts/session.md) をご覧ください。

## オプション

-   `--url `: Gateway WebSocket URL (設定されている場合は gateway.remote.url がデフォルト)。
-   `--token `: Gateway 認証トークン。
-   `--token-file `: ファイルから Gateway 認証トークンを読み込みます。
-   `--password `: Gateway 認証パスワード。
-   `--password-file `: ファイルから Gateway 認証パスワードを読み込みます。
-   `--session `: デフォルトのセッションキー。
-   `--session-label `: 解決するデフォルトのセッションラベル。
-   `--require-existing`: セッションキー/ラベルが存在しない場合に失敗します。
-   `--reset-session`: 最初の使用前にセッションキーをリセットします。
-   `--no-prefix-cwd`: プロンプトに作業ディレクトリをプレフィックスとして付けません。
-   `--verbose, -v`: stderr への詳細ログ出力。

セキュリティ上の注意:

-   `--token` と `--password` は、一部のシステムではローカルプロセス一覧で見える可能性があります。
-   `--token-file`/`--password-file` または環境変数 (`OPENCLAW_GATEWAY_TOKEN`, `OPENCLAW_GATEWAY_PASSWORD`) の使用を推奨します。
-   ACP ランタイムバックエンドの子プロセスは `OPENCLAW_SHELL=acp` を受け取り、コンテキスト固有のシェル/プロファイルルールに使用できます。
-   `openclaw acp client` は、起動されたブリッジプロセスに `OPENCLAW_SHELL=acp-client` を設定します。

### acp client オプション

-   `--cwd `: ACP セッションの作業ディレクトリ。
-   `--server `: ACP サーバーコマンド (デフォルト: `openclaw`)。
-   `--server-args <args...>`: ACP サーバーに渡す追加の引数。
-   `--server-verbose`: ACP サーバーで詳細ログを有効にします。
-   `--verbose, -v`: クライアントの詳細ログ出力。

[CLI リファレンス](../cli.md)[agent](./agent.md)