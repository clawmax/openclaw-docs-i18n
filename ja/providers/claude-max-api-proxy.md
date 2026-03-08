

  プロバイダー

  
# Claude Max API Proxy

**claude-max-api-proxy** は、あなたのClaude Max/ProサブスクリプションをOpenAI互換のAPIエンドポイントとして公開するコミュニティツールです。これにより、OpenAI API形式をサポートするあらゆるツールでサブスクリプションを利用できるようになります。

> **⚠️** この方法は技術的な互換性のみを提供します。Anthropicは過去にClaude Code以外での一部のサブスクリプション利用をブロックしたことがあります。これを使用するかどうかはご自身で判断し、依存する前にAnthropicの現在の利用規約を確認してください。

## なぜこれを使うのか？

| アプローチ | コスト | 最適な用途 |
| --- | --- | --- |
| Anthropic API | トークン単位の従量課金（Opusで約15/M入力, 75/M出力） | 本番アプリ、高ボリューム |
| Claude Max サブスクリプション | 月額$200固定 | 個人利用、開発、無制限使用 |

Claude Maxサブスクリプションをお持ちで、OpenAI互換ツールで使用したい場合、このプロキシは一部のワークフローでコスト削減につながる可能性があります。本番利用には、APIキーがより明確なポリシーの道筋となります。

## 仕組み

```
あなたのアプリ → claude-max-api-proxy → Claude Code CLI → Anthropic (サブスクリプション経由)
     (OpenAI形式)              (形式変換)      (あなたのログインを使用)
```

このプロキシは:

1.  `http://localhost:3456/v1/chat/completions` でOpenAI形式のリクエストを受け付けます
2.  それらをClaude Code CLIコマンドに変換します
3.  OpenAI形式でレスポンスを返します（ストリーミング対応）

## インストール

```bash
# Node.js 20+ と Claude Code CLI が必要です
npm install -g claude-max-api-proxy

# Claude CLIが認証済みか確認します
claude --version
```

## 使用方法

### サーバーを起動する

```
claude-max-api
# サーバーは http://localhost:3456 で実行されます
```

### テストする

```bash
# ヘルスチェック
curl http://localhost:3456/health

# モデル一覧を取得
curl http://localhost:3456/v1/models

# チャット補完
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### OpenClawで使用する

OpenClawをカスタムのOpenAI互換エンドポイントとしてこのプロキシに向けることができます:

```json
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```

## 利用可能なモデル

| モデルID | 対応モデル |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## macOSでの自動起動

LaunchAgentを作成してプロキシを自動的に実行します:

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## リンク

-   **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
-   **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
-   **Issues:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## 注意事項

-   これは **コミュニティツール** であり、AnthropicやOpenClawによる公式サポートはありません
-   アクティブなClaude Max/Proサブスクリプションと、認証済みのClaude Code CLIが必要です
-   プロキシはローカルで実行され、データをサードパーティのサーバーに送信しません
-   ストリーミングレスポンスは完全にサポートされています

## 関連項目

-   [Anthropicプロバイダー](./anthropic.md) - ClaudeのセットアップトークンまたはAPIキーを使用したOpenClawのネイティブ統合
-   [OpenAIプロバイダー](./openai.md) - OpenAI/Codexサブスクリプション用

[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)[Deepgram](./deepgram.md)

---