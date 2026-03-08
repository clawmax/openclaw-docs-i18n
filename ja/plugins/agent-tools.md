

  拡張機能

  
# プラグイン エージェントツール

OpenClawプラグインは、エージェント実行中にLLMに公開される**エージェントツール**（JSON‑schema関数）を登録できます。ツールは**必須**（常に利用可能）または**オプション**（オプトイン）に設定できます。エージェントツールは、メイン設定の `tools` セクション、またはエージェントごとの `agents.list[].tools` セクションで設定されます。許可リスト/拒否リストポリシーは、エージェントが呼び出せるツールを制御します。

## 基本ツール

```typescript
import { Type } from "@sinclair/typebox";

export default function (api) {
  api.registerTool({
    name: "my_tool",
    description: "Do a thing",
    parameters: Type.Object({
      input: Type.String(),
    }),
    async execute(_id, params) {
      return { content: [{ type: "text", text: params.input }] };
    },
  });
}
```

## オプションツール（オプトイン）

オプションツールは**決して**自動的に有効化されません。ユーザーはエージェントの許可リストに追加する必要があります。

```bash
export default function (api) {
  api.registerTool(
    {
      name: "workflow_tool",
      description: "Run a local workflow",
      parameters: {
        type: "object",
        properties: {
          pipeline: { type: "string" },
        },
        required: ["pipeline"],
      },
      async execute(_id, params) {
        return { content: [{ type: "text", text: params.pipeline }] };
      },
    },
    { optional: true },
  );
}
```

オプションツールを `agents.list[].tools.allow`（またはグローバルな `tools.allow`）で有効化します：

```json
{
  agents: {
    list: [
      {
        id: "main",
        tools: {
          allow: [
            "workflow_tool", // 特定のツール名
            "workflow", // プラグインID（そのプラグインの全ツールを有効化）
            "group:plugins", // 全プラグインツール
          ],
        },
      },
    ],
  },
}
```

ツールの可用性に影響するその他の設定項目：

-   プラグインツールのみを指定した許可リストは、プラグインのオプトインとして扱われます。コアツールは、許可リストにコアツールやグループも含めない限り有効なままです。
-   `tools.profile` / `agents.list[].tools.profile`（基本許可リスト）
-   `tools.byProvider` / `agents.list[].tools.byProvider`（プロバイダー固有の許可/拒否）
-   `tools.sandbox.tools.*`（サンドボックス化時のサンドボックストールポリシー）

## ルールとヒント

-   ツール名はコアツール名と**衝突してはなりません**。競合するツールはスキップされます。
-   許可リストで使用されるプラグインIDは、コアツール名と衝突してはなりません。
-   副作用を引き起こす、または追加のバイナリ/認証情報を必要とするツールには、`optional: true` を推奨します。

[プラグインマニフェスト](./manifest.md)[OpenProse](../prose.md)