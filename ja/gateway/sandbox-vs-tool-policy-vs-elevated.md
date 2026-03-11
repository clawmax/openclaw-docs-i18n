

  セキュリティとサンドボックス化

  
# サンドボックス vs ツールポリシー vs 高度モード

OpenClawには、関連するが異なる3つの制御があります：

1.  **サンドボックス** (`agents.defaults.sandbox.*` / `agents.list[].sandbox.*`) は、**ツールがどこで実行されるか**（Docker vs ホスト）を決定します。
2.  **ツールポリシー** (`tools.*`, `tools.sandbox.tools.*`, `agents.list[].tools.*`) は、**どのツールが利用可能/許可されるか**を決定します。
3.  **高度モード** (`tools.elevated.*`, `agents.list[].tools.elevated.*`) は、サンドボックス化されているときにホスト上で実行するための **exec専用のエスケープハッチ** です。

## クイックデバッグ

インスペクタを使用して、OpenClawが*実際に*何をしているかを確認します：

```bash
openclaw sandbox explain
openclaw sandbox explain --session agent:main:main
openclaw sandbox explain --agent work
openclaw sandbox explain --json
```

以下の内容を出力します：

-   有効なサンドボックスモード/スコープ/ワークスペースアクセス
-   セッションが現在サンドボックス化されているかどうか（メイン vs 非メイン）
-   有効なサンドボックスのツール許可/拒否（およびそれがエージェント/グローバル/デフォルトから来たものかどうか）
-   高度モードのゲートと修正キーのパス

## サンドボックス: ツールが実行される場所

サンドボックス化は `agents.defaults.sandbox.mode` によって制御されます：

-   `"off"`: すべてがホスト上で実行されます。
-   `"non-main"`: 非メインセッションのみがサンドボックス化されます（グループ/チャネルでの一般的な「驚き」）。
-   `"all"`: すべてがサンドボックス化されます。

完全なマトリックス（スコープ、ワークスペースマウント、イメージ）については、[サンドボックス化](./sandboxing.md)を参照してください。

### バインドマウント（セキュリティクイックチェック）

-   `docker.binds` はサンドボックスのファイルシステムを*貫通*します：マウントしたものは、設定したモード（`:ro` または `:rw`）でコンテナ内から見えます。
-   モードを省略した場合のデフォルトは読み書き可能です。ソース/シークレットには `:ro` を推奨します。
-   `scope: "shared"` はエージェントごとのバインドを無視します（グローバルバインドのみが適用されます）。
-   `/var/run/docker.sock` をバインドすると、事実上ホストの制御をサンドボックスに渡します。意図的にのみ行ってください。
-   ワークスペースアクセス (`workspaceAccess: "ro"`/`"rw"`) はバインドモードとは独立しています。

## ツールポリシー: どのツールが存在するか/呼び出し可能か

2つのレイヤーが重要です：

-   **ツールプロファイル**: `tools.profile` と `agents.list[].tools.profile`（基本許可リスト）
-   **プロバイダーツールプロファイル**: `tools.byProvider[provider].profile` と `agents.list[].tools.byProvider[provider].profile`
-   **グローバル/エージェントごとのツールポリシー**: `tools.allow`/`tools.deny` と `agents.list[].tools.allow`/`agents.list[].tools.deny`
-   **プロバイダーツールポリシー**: `tools.byProvider[provider].allow/deny` と `agents.list[].tools.byProvider[provider].allow/deny`
-   **サンドボックスのツールポリシー**（サンドボックス化されている場合のみ適用）: `tools.sandbox.tools.allow`/`tools.sandbox.tools.deny` と `agents.list[].tools.sandbox.tools.*`

経験則：

-   `deny` が常に優先されます。
-   `allow` が空でない場合、他のすべてはブロックされたものとして扱われます。
-   ツールポリシーはハードストップです：拒否された `exec` ツールを `/exec` で上書きすることはできません。
-   `/exec` は、許可された送信者に対するセッションのデフォルトを変更するだけで、ツールへのアクセスを付与しません。プロバイダーツールキーは、`provider`（例：`google-antigravity`）または `provider/model`（例：`openai/gpt-5.2`）のいずれかを受け入れます。

### ツールグループ（短縮形）

ツールポリシー（グローバル、エージェント、サンドボックス）は、複数のツールに展開される `group:*` エントリをサポートします：

```json
{
  tools: {
    sandbox: {
      tools: {
        allow: ["group:runtime", "group:fs", "group:sessions", "group:memory"],
      },
    },
  },
}
```

利用可能なグループ：

-   `group:runtime`: `exec`, `bash`, `process`
-   `group:fs`: `read`, `write`, `edit`, `apply_patch`
-   `group:sessions`: `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`, `session_status`
-   `group:memory`: `memory_search`, `memory_get`
-   `group:ui`: `browser`, `canvas`
-   `group:automation`: `cron`, `gateway`
-   `group:messaging`: `message`
-   `group:nodes`: `nodes`
-   `group:openclaw`: すべての組み込みOpenClawツール（プロバイダープラグインを除く）

## 高度モード: exec専用の「ホスト上で実行」

高度モードは**追加のツールを付与しません**。`exec` にのみ影響します。

-   サンドボックス化されている場合、`/elevated on`（または `elevated: true` を指定した `exec`）はホスト上で実行されます（承認がまだ適用される場合があります）。
-   セッションのexec承認をスキップするには `/elevated full` を使用します。
-   すでに直接実行している場合、高度モードは事実上何もしません（ゲートは適用されます）。
-   高度モードは**スキルスコープではなく**、ツールの許可/拒否を**上書きしません**。
-   `/exec` は高度モードとは別です。許可された送信者に対して、セッションごとのexecデフォルトを調整するだけです。

ゲート：

-   有効化: `tools.elevated.enabled`（およびオプションで `agents.list[].tools.elevated.enabled`）
-   送信者許可リスト: `tools.elevated.allowFrom.`（およびオプションで `agents.list[].tools.elevated.allowFrom.`）

[高度モード](../tools/elevated.md)を参照してください。

## 一般的な「サンドボックス監獄」の修正

### 「ツールXがサンドボックスのツールポリシーによってブロックされました」

修正キー（いずれか1つを選択）：

-   サンドボックスを無効化: `agents.defaults.sandbox.mode=off`（またはエージェントごとに `agents.list[].sandbox.mode=off`）
-   サンドボックス内でツールを許可：
    -   `tools.sandbox.tools.deny`（またはエージェントごとの `agents.list[].tools.sandbox.tools.deny`）から削除する
    -   または `tools.sandbox.tools.allow`（またはエージェントごとのallow）に追加する

### 「これはメインだと思っていましたが、なぜサンドボックス化されているのですか？」

`"non-main"` モードでは、グループ/チャネルキーは*メインではありません*。メインセッションキー（`sandbox explain` で表示）を使用するか、モードを `"off"` に切り替えてください。

[サンドボックス化](./sandboxing.md)[ゲートウェイプロトコル](./protocol.md)