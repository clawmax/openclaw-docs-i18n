

  実験機能

  
# オンボーディングと設定プロトコル

目的: CLI、macOS アプリ、Web UI にわたる共通のオンボーディング + 設定インターフェース。

## コンポーネント

-   ウィザードエンジン (共有セッション + プロンプト + オンボーディング状態)。
-   CLI オンボーディングは UI クライアントと同じウィザードフローを使用します。
-   ゲートウェイ RPC はウィザード + 設定スキーマエンドポイントを公開します。
-   macOS オンボーディングはウィザードステップモデルを使用します。
-   Web UI は JSON スキーマ + UI ヒントから設定フォームをレンダリングします。

## ゲートウェイ RPC

-   `wizard.start` パラメータ: `{ mode?: "local"|"remote", workspace?: string }`
-   `wizard.next` パラメータ: `{ sessionId, answer?: { stepId, value? } }`
-   `wizard.cancel` パラメータ: `{ sessionId }`
-   `wizard.status` パラメータ: `{ sessionId }`
-   `config.schema` パラメータ: `{}`
-   `config.schema.lookup` パラメータ: `{ path }`
    -   `path` は標準の設定セグメントに加え、スラッシュ区切りのプラグイン ID を受け付けます。例: `plugins.entries.pack/one.config`。

レスポンス (形式)

-   ウィザード: `{ sessionId, done, step?, status?, error? }`
-   設定スキーマ: `{ schema, uiHints, version, generatedAt }`
-   設定スキーマルックアップ: `{ path, schema, hint?, hintPath?, children[] }`

## UI ヒント

-   `uiHints` はパスでキー指定されるオプションのメタデータ (label/help/group/order/advanced/sensitive/placeholder)。
-   センシティブなフィールドはパスワード入力としてレンダリングされます。編集不可レイヤーはありません。
-   サポートされていないスキーマノードは生の JSON エディタにフォールバックします。

## 注記

-   このドキュメントは、オンボーディング/設定のプロトコルリファクタリングを追跡する唯一の場所です。

[Kilo ゲートウェイ統合](../design/kilo-gateway-integration.md)[ACP スレッドバインドエージェント](./plans/acp-thread-bound-agents.md)