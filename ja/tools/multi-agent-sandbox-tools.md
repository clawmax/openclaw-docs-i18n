

  エージェント連携

  
# マルチエージェントサンドボックス & ツール

## 概要

マルチエージェント設定において、各エージェントは独自の以下を持つことができます:

-   **サンドボックス設定** (`agents.list[].sandbox` が `agents.defaults.sandbox` を上書き)
-   **ツール制限** (`tools.allow` / `tools.deny`、および `agents.list[].tools`)

これにより、異なるセキュリティプロファイルを持つ複数のエージェントを実行できます:

-   フルアクセス権を持つ個人用アシスタント
-   制限されたツールを持つ家族/仕事用エージェント
-   サンドボックス内の公開向けエージェント

`setupCommand` は `sandbox.docker` の下（グローバルまたはエージェントごと）に属し、コンテナ作成時に一度実行されます。認証はエージェントごとです: 各エージェントは自身の `agentDir` 認証ストアから読み取ります:

```
~/.openclaw/agents/<agentId>/agent/auth-profiles.json
```

認証情報はエージェント間で**共有されません**。`agentDir` をエージェント間で再利用しないでください。認証情報を共有したい場合は、`auth-profiles.json` を他のエージェントの `agentDir` にコピーしてください。サンドボックス化の実行時の動作については、[サンドボックス化](../gateway/sandboxing.md)を参照してください。「なぜこれがブロックされるのか？」のデバッグについては、[サンドボックス vs ツールポリシー vs 昇格](../gateway/sandbox-vs-tool-policy-vs-elevated.md)と `openclaw sandbox explain` を参照してください。

* * *

## 設定例

### 例1: 個人用 + 制限付き家族用エージェント

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "name": "個人用アシスタント",
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "family",
        "name": "家族用ボット",
        "workspace": "~/.openclaw/workspace-family",
        "sandbox": {
          "mode": "all",
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch", "process", "browser"]
        }
      }
    ]
  },
  "bindings": [
    {
      "agentId": "family",
      "match": {
        "provider": "whatsapp",
        "accountId": "*",
        "peer": {
          "kind": "group",
          "id": "120363424282127706@g.us"
        }
      }
    }
  ]
}
```

**結果:**

-   `main` エージェント: ホスト上で実行、ツールへのフルアクセス
-   `family` エージェント: Docker内で実行（エージェントごとに1コンテナ）、`read` ツールのみ

* * *

### 例2: 共有サンドボックスを持つ仕事用エージェント

```json
{
  "agents": {
    "list": [
      {
        "id": "personal",
        "workspace": "~/.openclaw/workspace-personal",
        "sandbox": { "mode": "off" }
      },
      {
        "id": "work",
        "workspace": "~/.openclaw/workspace-work",
        "sandbox": {
          "mode": "all",
          "scope": "shared",
          "workspaceRoot": "/tmp/work-sandboxes"
        },
        "tools": {
          "allow": ["read", "write", "apply_patch", "exec"],
          "deny": ["browser", "gateway", "discord"]
        }
      }
    ]
  }
}
```

* * *

### 例2b: グローバルコーディングプロファイル + メッセージング専用エージェント

```json
{
  "tools": { "profile": "coding" },
  "agents": {
    "list": [
      {
        "id": "support",
        "tools": { "profile": "messaging", "allow": ["slack"] }
      }
    ]
  }
}
```

**結果:**

-   デフォルトエージェントはコーディングツールを取得
-   `support` エージェントはメッセージング専用 (+ Slackツール)

* * *

### 例3: エージェントごとの異なるサンドボックスモード

```json
{
  "agents": {
    "defaults": {
      "sandbox": {
        "mode": "non-main", // グローバルデフォルト
        "scope": "session"
      }
    },
    "list": [
      {
        "id": "main",
        "workspace": "~/.openclaw/workspace",
        "sandbox": {
          "mode": "off" // 上書き: mainはサンドボックス化されない
        }
      },
      {
        "id": "public",
        "workspace": "~/.openclaw/workspace-public",
        "sandbox": {
          "mode": "all", // 上書き: publicは常にサンドボックス化
          "scope": "agent"
        },
        "tools": {
          "allow": ["read"],
          "deny": ["exec", "write", "edit", "apply_patch"]
        }
      }
    ]
  }
}
```

* * *

## 設定の優先順位

グローバル (`agents.defaults.*`) とエージェント固有 (`agents.list[].*`) の両方の設定が存在する場合:

### サンドボックス設定

エージェント固有の設定がグローバルを上書きします:

```
agents.list[].sandbox.mode > agents.defaults.sandbox.mode
agents.list[].sandbox.scope > agents.defaults.sandbox.scope
agents.list[].sandbox.workspaceRoot > agents.defaults.sandbox.workspaceRoot
agents.list[].sandbox.workspaceAccess > agents.defaults.sandbox.workspaceAccess
agents.list[].sandbox.docker.* > agents.defaults.sandbox.docker.*
agents.list[].sandbox.browser.* > agents.defaults.sandbox.browser.*
agents.list[].sandbox.prune.* > agents.defaults.sandbox.prune.*
```

**注意:**

-   `agents.list[].sandbox.{docker,browser,prune}.*` は、そのエージェントに対して `agents.defaults.sandbox.{docker,browser,prune}.*` を上書きします（サンドボックススコープが `"shared"` に解決される場合は無視されます）。

### ツール制限

フィルタリング順序は:

1.  **ツールプロファイル** (`tools.profile` または `agents.list[].tools.profile`)
2.  **プロバイダーツールプロファイル** (`tools.byProvider[provider].profile` または `agents.list[].tools.byProvider[provider].profile`)
3.  **グローバルツールポリシー** (`tools.allow` / `tools.deny`)
4.  **プロバイダーツールポリシー** (`tools.byProvider[provider].allow/deny`)
5.  **エージェント固有ツールポリシー** (`agents.list[].tools.allow/deny`)
6.  **エージェントプロバイダーポリシー** (`agents.list[].tools.byProvider[provider].allow/deny`)
7.  **サンドボックスのツールポリシー** (`tools.sandbox.tools` または `agents.list[].tools.sandbox.tools`)
8.  **サブエージェントのツールポリシー** (`tools.subagents.tools`、該当する場合)

各レベルはツールをさらに制限できますが、前のレベルで拒否されたツールを許可することはできません。`agents.list[].tools.sandbox.tools` が設定されている場合、そのエージェントに対して `tools.sandbox.tools` を置き換えます。`agents.list[].tools.profile` が設定されている場合、そのエージェントに対して `tools.profile` を上書きします。プロバイダーツールキーは、`provider`（例: `google-antigravity`）または `provider/model`（例: `openai/gpt-5.2`）のいずれかを受け入れます。

### ツールグループ（短縮形）

ツールポリシー（グローバル、エージェント、サンドボックス）は、複数の具体的なツールに展開される `group:*` エントリをサポートします:

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: すべての組み込みOpenClawツール（プロバイダープラグインを除く）

### 昇格モード

`tools.elevated` はグローバルベースライン（送信者ベースの許可リスト）です。`agents.list[].tools.elevated` は特定のエージェントに対して昇格をさらに制限できます（両方が許可する必要があります）。緩和パターン:

-   信頼できないエージェントに対して `exec` を拒否 (`agents.list[].tools.deny: ["exec"]`)
-   制限付きエージェントにルーティングする送信者の許可リスト化を避ける
-   サンドボックス化された実行のみが必要な場合は、グローバルで昇格を無効化 (`tools.elevated.enabled: false`)
-   機密性の高いプロファイルに対してエージェントごとに昇格を無効化 (`agents.list[].tools.elevated.enabled: false`)

* * *

## 単一エージェントからの移行

**以前（単一エージェント）:**

```json
{
  "agents": {
    "defaults": {
      "workspace": "~/.openclaw/workspace",
      "sandbox": {
        "mode": "non-main"
      }
    }
  },
  "tools": {
    "sandbox": {
      "tools": {
        "allow": ["read", "write", "apply_patch", "exec"],
        "deny": []
      }
    }
  }
}
```

**以降（異なるプロファイルを持つマルチエージェント）:**

```json
{
  "agents": {
    "list": [
      {
        "id": "main",
        "default": true,
        "workspace": "~/.openclaw/workspace",
        "sandbox": { "mode": "off" }
      }
    ]
  }
}
```

従来の `agent.*` 設定は `openclaw doctor` によって移行されます。今後は `agents.defaults` + `agents.list` を推奨します。

* * *

## ツール制限の例

### 読み取り専用エージェント

```json
{
  "tools": {
    "allow": ["read"],
    "deny": ["exec", "write", "edit", "apply_patch", "process"]
  }
}
```

### 安全な実行エージェント（ファイル変更なし）

```json
{
  "tools": {
    "allow": ["read", "exec", "process"],
    "deny": ["write", "edit", "apply_patch", "browser", "gateway"]
  }
}
```

### 通信専用エージェント

```json
{
  "tools": {
    "sessions": { "visibility": "tree" },
    "allow": ["sessions_list", "sessions_send", "sessions_history", "session_status"],
    "deny": ["exec", "write", "edit", "apply_patch", "read", "browser"]
  }
}
```

* * *

## よくある落とし穴: "non-main"

`agents.defaults.sandbox.mode: "non-main"` はエージェントIDではなく、`session.mainKey`（デフォルト `"main"`）に基づいています。グループ/チャネルセッションは常に独自のキーを持つため、non-mainとして扱われ、サンドボックス化されます。エージェントを決してサンドボックス化したくない場合は、`agents.list[].sandbox.mode: "off"` を設定してください。

* * *

## テスト

マルチエージェントサンドボックスとツールを設定した後:

1.  **エージェント解決を確認:**
    
    コピー
    
    ```bash
    openclaw agents list --bindings
    ```
    
2.  **サンドボックスコンテナを確認:**
    
    コピー
    
    ```bash
    docker ps --filter "name=openclaw-sbx-"
    ```
    
3.  **ツール制限をテスト:**
    -   制限されたツールを必要とするメッセージを送信
    -   エージェントが拒否されたツールを使用できないことを確認
4.  **ログを監視:**
    
    コピー
    
    ```bash
    tail -f "${OPENCLAW_STATE_DIR:-$HOME/.openclaw}/logs/gateway.log" | grep -E "routing|sandbox|tools"
    ```
    

* * *

## トラブルシューティング

### モードが "all" でもエージェントがサンドボックス化されない

-   それを上書きするグローバルな `agents.defaults.sandbox.mode` がないか確認
-   エージェント固有の設定が優先されるため、`agents.list[].sandbox.mode: "all"` を設定

### 拒否リストにもかかわらずツールが利用可能

-   ツールフィルタリング順序を確認: グローバル → エージェント → サンドボックス → サブエージェント
-   各レベルはさらに制限することしかできず、許可を取り戻すことはできない
-   ログで確認: `[tools] filtering tools for agent:${agentId}`

### コンテナがエージェントごとに分離されていない

-   エージェント固有のサンドボックス設定で `scope: "agent"` を設定
-   デフォルトは `"session"` で、セッションごとに1つのコンテナを作成

* * *

## 関連項目

-   [マルチエージェントルーティング](../concepts/multi-agent.md)
-   [サンドボックス設定](../gateway/configuration.md#agentsdefaults-sandbox)
-   [セッション管理](../concepts/session.md)

[ACPエージェント](./acp-agents.md)[スキルの作成](./creating-skills.md)