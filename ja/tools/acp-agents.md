title: "外部コーディングハーネス向け OpenClaw ACP エージェントガイド"
description: "OpenClaw ACP エージェントを使用して Claude Code や Codex などの外部コーディングハーネスを実行する方法を学びます。セッションの設定、ランタイムの管理、acpx バックエンドのセットアップについて説明します。"
keywords: ["acp エージェント", "エージェントクライアントプロトコル", "openclaw acp", "acpx バックエンド", "コーディングハーネス", "claude code", "codex", "エージェント調整"]
---

  エージェント調整

  
# ACP エージェント

[エージェントクライアントプロトコル (ACP)](https://agentclientprotocol.com/) セッションにより、OpenClaw は ACP バックエンドプラグインを通じて外部コーディングハーネス（例: Pi、Claude Code、Codex、OpenCode、Gemini CLI）を実行できます。OpenClaw に対して「これを Codex で実行して」や「スレッドで Claude Code を開始して」と平文で依頼すると、OpenClaw はそのリクエストを ACP ランタイム（ネイティブのサブエージェントランタイムではなく）にルーティングするはずです。

## オペレーター向け高速フロー

実用的な `/acp` 実行手順が必要な場合はこちらを使用してください:

1.  セッションを生成:
    -   `/acp spawn codex --mode persistent --thread auto`
2.  バインドされたスレッドで作業（またはそのセッションキーを明示的にターゲット）。
3.  ランタイム状態を確認:
    -   `/acp status`
4.  必要に応じてランタイムオプションを調整:
    -   `/acp model <provider/model>`
    -   `/acp permissions `
    -   `/acp timeout `
5.  コンテキストを置き換えずにアクティブなセッションを微調整:
    -   `/acp steer tighten logging and continue`
6.  作業を停止:
    -   `/acp cancel` (現在のターンを停止)、または
    -   `/acp close` (セッションを閉じる + バインディングを削除)

## 人間向けクイックスタート

自然なリクエストの例:

-   「ここで永続的な Codex セッションをスレッドで開始し、フォーカスを維持してください。」
-   「これをワンショットの Claude Code ACP セッションとして実行し、結果を要約してください。」
-   「このタスクに Gemini CLI を使用し、その後のフォローアップは同じスレッドで行ってください。」

OpenClaw が行うこと:

1.  `runtime: "acp"` を選択。
2.  要求されたハーネスターゲット (`agentId`、例: `codex`) を解決。
3.  スレッドバインディングが要求され、現在のチャネルがそれをサポートしている場合、ACP セッションをスレッドにバインド。
4.  フォローアップのスレッドメッセージを、フォーカスが外れる/閉じられる/期限切れになるまで同じ ACP セッションにルーティング。

## ACP 対 サブエージェント

外部ハーネスランタイムが必要な場合は ACP を使用。OpenClaw ネイティブの委任実行が必要な場合はサブエージェントを使用。

| 領域 | ACP セッション | サブエージェント実行 |
| --- | --- | --- |
| ランタイム | ACP バックエンドプラグイン (例: acpx) | OpenClaw ネイティブサブエージェントランタイム |
| セッションキー | `agent::acp:` | `agent::subagent:` |
| 主要コマンド | `/acp ...` | `/subagents ...` |
| 生成ツール | `sessions_spawn` with `runtime:"acp"` | `sessions_spawn` (デフォルトランタイム) |

[サブエージェント](./subagents.md)も参照してください。

## スレッドバインドセッション (チャネル非依存)

チャネルアダプターでスレッドバインディングが有効になっている場合、ACP セッションをスレッドにバインドできます:

-   OpenClaw はスレッドをターゲット ACP セッションにバインドします。
-   そのスレッド内のフォローアップメッセージはバインドされた ACP セッションにルーティングされます。
-   ACP 出力は同じスレッドに返送されます。
-   フォーカス解除/閉じる/アーカイブ/アイドルタイムアウトまたは最大経過時間によりバインディングが削除されます。

スレッドバインディングのサポートはアダプター固有です。アクティブなチャネルアダプターがスレッドバインディングをサポートしていない場合、OpenClaw は明確な未サポート/利用不可メッセージを返します。スレッドバインド ACP に必要な機能フラグ:

-   `acp.enabled=true`
-   `acp.dispatch.enabled` はデフォルトでオン (ACP ディスパッチを一時停止するには `false` に設定)
-   チャネルアダプターの ACP スレッド生成フラグが有効 (アダプター固有)
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

### スレッドをサポートするチャネル

-   セッション/スレッドバインディング機能を公開する任意のチャネルアダプター。
-   現在の組み込みサポート:
    -   Discord スレッド/チャネル
    -   Telegram トピック (グループ/スーパーグループ内のフォーラムトピックおよび DM トピック)
-   プラグインチャネルは同じバインディングインターフェースを通じてサポートを追加できます。

## チャネル固有の設定

非一時的なワークフローの場合、トップレベルの `bindings[]` エントリで永続的な ACP バインディングを構成します。

### バインディングモデル

-   `bindings[].type="acp"` は永続的な ACP 会話バインディングを示します。
-   `bindings[].match` はターゲット会話を識別:
    -   Discord チャネルまたはスレッド: `match.channel="discord"` + `match.peer.id=""`
    -   Telegram フォーラムトピック: `match.channel="telegram"` + `match.peer.id=":topic:"`
-   `bindings[].agentId` は所有する OpenClaw エージェント ID です。
-   オプションの ACP オーバーライドは `bindings[].acp` の下に配置:
    -   `mode` (`persistent` または `oneshot`)
    -   `label`
    -   `cwd`
    -   `backend`

### エージェントごとのランタイムデフォルト

`agents.list[].runtime` を使用してエージェントごとに ACP デフォルトを定義:

-   `agents.list[].runtime.type="acp"`
-   `agents.list[].runtime.acp.agent` (ハーネス ID、例: `codex` または `claude`)
-   `agents.list[].runtime.acp.backend`
-   `agents.list[].runtime.acp.mode`
-   `agents.list[].runtime.acp.cwd`

ACP バインドセッションのオーバーライド優先順位:

1.  `bindings[].acp.*`
2.  `agents.list[].runtime.acp.*`
3.  グローバル ACP デフォルト (例: `acp.backend`)

例:

```json
{
  agents: {
    list: [
      {
        id: "codex",
        runtime: {
          type: "acp",
          acp: {
            agent: "codex",
            backend: "acpx",
            mode: "persistent",
            cwd: "/workspace/openclaw",
          },
        },
      },
      {
        id: "claude",
        runtime: {
          type: "acp",
          acp: { agent: "claude", backend: "acpx", mode: "persistent" },
        },
      },
    ],
  },
  bindings: [
    {
      type: "acp",
      agentId: "codex",
      match: {
        channel: "discord",
        accountId: "default",
        peer: { kind: "channel", id: "222222222222222222" },
      },
      acp: { label: "codex-main" },
    },
    {
      type: "acp",
      agentId: "claude",
      match: {
        channel: "telegram",
        accountId: "default",
        peer: { kind: "group", id: "-1001234567890:topic:42" },
      },
      acp: { cwd: "/workspace/repo-b" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "discord", accountId: "default" },
    },
    {
      type: "route",
      agentId: "main",
      match: { channel: "telegram", accountId: "default" },
    },
  ],
  channels: {
    discord: {
      guilds: {
        "111111111111111111": {
          channels: {
            "222222222222222222": { requireMention: false },
          },
        },
      },
    },
    telegram: {
      groups: {
        "-1001234567890": {
          topics: { "42": { requireMention: false } },
        },
      },
    },
  },
}
```

動作:

-   OpenClaw は使用前に構成された ACP セッションが存在することを保証します。
-   そのチャネルまたはトピック内のメッセージは構成された ACP セッションにルーティングされます。
-   バインドされた会話では、`/new` と `/reset` は同じ ACP セッションキーをその場でリセットします。
-   一時的なランタイムバインディング (例: スレッドフォーカスフローによって作成されたもの) は、存在する場合に引き続き適用されます。

## ACP セッションの開始 (インターフェース)

### sessions_spawn から

エージェントターンまたはツール呼び出しから ACP セッションを開始するには `runtime: "acp"` を使用します。

```json
{
  "task": "Open the repo and summarize failing tests",
  "runtime": "acp",
  "agentId": "codex",
  "thread": true,
  "mode": "session"
}
```

注意:

-   `runtime` はデフォルトで `subagent` なので、ACP セッションには明示的に `runtime: "acp"` を設定します。
-   `agentId` が省略された場合、OpenClaw は構成時に `acp.defaultAgent` を使用します。
-   `mode: "session"` には永続的なバインド会話を維持するために `thread: true` が必要です。

インターフェース詳細:

-   `task` (必須): ACP セッションに送信される初期プロンプト。
-   `runtime` (ACP に必須): `"acp"` でなければなりません。
-   `agentId` (オプション): ACP ターゲットハーネス ID。設定されている場合は `acp.defaultAgent` にフォールバック。
-   `thread` (オプション、デフォルト `false`): サポートされている場合、スレッドバインディングフローを要求。
-   `mode` (オプション): `run` (ワンショット) または `session` (永続的)。
    -   デフォルトは `run`
    -   `thread: true` で mode が省略された場合、OpenClaw はランタイムパスごとに永続的な動作をデフォルトとする場合があります
    -   `mode: "session"` には `thread: true` が必要
-   `cwd` (オプション): 要求されたランタイム作業ディレクトリ (バックエンド/ランタイムポリシーによって検証)。
-   `label` (オプション): オペレーター向けラベル、セッション/バナーテキストで使用。
-   `streamTo` (オプション): `"parent"` は初期 ACP 実行の進捗概要をシステムイベントとして要求元セッションにストリーム返送します。
    -   利用可能な場合、受け入れられた応答には、完全なリレー履歴を tail できるセッションスコープの JSONL ログ (`.acp-stream.jsonl`) を指す `streamLogPath` が含まれます。

## サンドボックス互換性

ACP セッションは現在、OpenClaw サンドボックス内ではなく、ホストランタイム上で実行されます。現在の制限:

-   要求元セッションがサンドボックス化されている場合、ACP 生成はブロックされます。
    -   エラー: `Sandboxed sessions cannot spawn ACP sessions because runtime="acp" runs on the host. Use runtime="subagent" from sandboxed sessions.`
-   `sessions_spawn` で `runtime: "acp"` の場合、`sandbox: "require"` はサポートされません。
    -   エラー: `sessions_spawn sandbox="require" is unsupported for runtime="acp" because ACP sessions run outside the sandbox. Use runtime="subagent" or sandbox="inherit".`

サンドボックス強制実行が必要な場合は `runtime: "subagent"` を使用してください。

### /acp コマンドから

必要に応じてチャットから明示的なオペレーター制御を行うには `/acp spawn` を使用します。

```bash
/acp spawn codex --mode persistent --thread auto
/acp spawn codex --mode oneshot --thread off
/acp spawn codex --thread here
```

主要フラグ:

-   `--mode persistent|oneshot`
-   `--thread auto|here|off`
-   `--cwd <absolute-path>`
-   `--label `

[スラッシュコマンド](./slash-commands.md)を参照してください。

## セッションターゲット解決

ほとんどの `/acp` アクションはオプションのセッションターゲット (`session-key`、`session-id`、`session-label`) を受け入れます。解決順序:

1.  明示的なターゲット引数 (または `/acp steer` の `--session`)
    -   キーを試行
    -   次に UUID 形式のセッション ID
    -   次にラベル
2.  現在のスレッドバインディング (この会話/スレッドが ACP セッションにバインドされている場合)
3.  現在の要求元セッションへのフォールバック

ターゲットが解決されない場合、OpenClaw は明確なエラーを返します (`Unable to resolve session target: ...`)。

## スレッド生成モード

`/acp spawn` は `--thread auto|here|off` をサポートします。

| モード | 動作 |
| --- | --- |
| `auto` | アクティブなスレッド内: そのスレッドをバインド。スレッド外: サポートされている場合、子スレッドを作成/バインド。 |
| `here` | 現在のアクティブなスレッドを要求; ない場合は失敗。 |
| `off` | バインディングなし。セッションはバインドされずに開始。 |

注意:

-   非スレッドバインディングサーフェスでは、デフォルトの動作は実質的に `off` です。
-   スレッドバインド生成にはチャネルポリシーのサポートが必要:
    -   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`
    -   Telegram: `channels.telegram.threadBindings.spawnAcpSessions=true`

## ACP コントロール

利用可能なコマンドファミリー:

-   `/acp spawn`
-   `/acp cancel`
-   `/acp steer`
-   `/acp close`
-   `/acp status`
-   `/acp set-mode`
-   `/acp set`
-   `/acp cwd`
-   `/acp permissions`
-   `/acp timeout`
-   `/acp model`
-   `/acp reset-options`
-   `/acp sessions`
-   `/acp doctor`
-   `/acp install`

`/acp status` は有効なランタイムオプションと、利用可能な場合はランタイムレベルとバックエンドレベルの両方のセッション識別子を表示します。一部のコントロールはバックエンドの機能に依存します。バックエンドがコントロールをサポートしていない場合、OpenClaw は明確な未サポートコントロールエラーを返します。

## ACP コマンドクックブック

| コマンド | 機能 | 例 |
| --- | --- | --- |
| `/acp spawn` | ACP セッションを作成; オプションでスレッドバインド。 | `/acp spawn codex --mode persistent --thread auto --cwd /repo` |
| `/acp cancel` | ターゲットセッションの実行中のターンをキャンセル。 | `/acp cancel agent:codex:acp:` |
| `/acp steer` | 実行中のセッションに指示を送信。 | `/acp steer --session support inbox prioritize failing tests` |
| `/acp close` | セッションを閉じ、スレッドターゲットのバインドを解除。 | `/acp close` |
| `/acp status` | バックエンド、モード、状態、ランタイムオプション、機能を表示。 | `/acp status` |
| `/acp set-mode` | ターゲットセッションのランタイムモードを設定。 | `/acp set-mode plan` |
| `/acp set` | 汎用ランタイム設定オプション書き込み。 | `/acp set model openai/gpt-5.2` |
| `/acp cwd` | ランタイム作業ディレクトリのオーバーライドを設定。 | `/acp cwd /Users/user/Projects/repo` |
| `/acp permissions` | 承認ポリシープロファイルを設定。 | `/acp permissions strict` |
| `/acp timeout` | ランタイムタイムアウト (秒) を設定。 | `/acp timeout 120` |
| `/acp model` | ランタイムモデルオーバーライドを設定。 | `/acp model anthropic/claude-opus-4-5` |
| `/acp reset-options` | セッションのランタイムオプションオーバーライドを削除。 | `/acp reset-options` |
| `/acp sessions` | ストアから最近の ACP セッションを一覧表示。 | `/acp sessions` |
| `/acp doctor` | バックエンドの健全性、機能、実行可能な修正。 | `/acp doctor` |
| `/acp install` | 決定論的なインストールと有効化の手順を表示。 | `/acp install` |

## ランタイムオプションマッピング

`/acp` には便利なコマンドと汎用セッターがあります。同等の操作:

-   `/acp model ` はランタイム設定キー `model` にマップ。
-   `/acp permissions ` はランタイム設定キー `approval_policy` にマップ。
-   `/acp timeout ` はランタイム設定キー `timeout` にマップ。
-   `/acp cwd ` はランタイム cwd オーバーライドを直接更新。
-   `/acp set  ` は汎用パス。
    -   特殊ケース: `key=cwd` は cwd オーバーライドパスを使用。
-   `/acp reset-options` はターゲットセッションのすべてのランタイムオーバーライドをクリア。

## acpx ハーネスサポート (現在)

現在の acpx 組み込みハーネスエイリアス:

-   `pi`
-   `claude`
-   `codex`
-   `opencode`
-   `gemini`
-   `kimi`

OpenClaw が acpx バックエンドを使用する場合、acpx 設定がカスタムエージェントエイリアスを定義していない限り、`agentId` にはこれらの値を優先してください。直接の acpx CLI 使用では `--agent ` を介して任意のアダプターをターゲットにすることもできますが、その生のエスケープハッチは acpx CLI 機能です (通常の OpenClaw `agentId` パスではありません)。

## 必要な設定

コア ACP ベースライン:

```json
{
  acp: {
    enabled: true,
    // オプション。デフォルトは true; /acp コントロールを維持しながら ACP ディスパッチを一時停止するには false に設定。
    dispatch: { enabled: true },
    backend: "acpx",
    defaultAgent: "codex",
    allowedAgents: ["pi", "claude", "codex", "opencode", "gemini", "kimi"],
    maxConcurrentSessions: 8,
    stream: {
      coalesceIdleMs: 300,
      maxChunkChars: 1200,
    },
    runtime: {
      ttlMinutes: 120,
    },
  },
}
```

スレッドバインディング設定はチャネルアダプター固有です。Discord の例:

```json
{
  session: {
    threadBindings: {
      enabled: true,
      idleHours: 24,
      maxAgeHours: 0,
    },
  },
  channels: {
    discord: {
      threadBindings: {
        enabled: true,
        spawnAcpSessions: true,
      },
    },
  },
}
```

スレッドバインド ACP 生成が機能しない場合は、まずアダプター機能フラグを確認してください:

-   Discord: `channels.discord.threadBindings.spawnAcpSessions=true`

[設定リファレンス](../gateway/configuration-reference.md)を参照してください。

## acpx バックエンドのプラグインセットアップ

プラグインをインストールして有効化:

```bash
openclaw plugins install acpx
openclaw config set plugins.entries.acpx.enabled true
```

開発中のローカルワークスペースインストール:

```bash
openclaw plugins install ./extensions/acpx
```

その後、バックエンドの健全性を確認:

```bash
/acp doctor
```

### acpx コマンドとバージョン設定

デフォルトでは、acpx プラグイン (公開名 `@openclaw/acpx`) はプラグインローカルに固定されたバイナリを使用します:

1.  コマンドはデフォルトで `extensions/acpx/node_modules/.bin/acpx`。
2.  期待されるバージョンはデフォルトで拡張機能の固定バージョン。
3.  起動時に ACP バックエンドがすぐに not-ready として登録されます。
4.  バックグラウンドの ensure ジョブが `acpx --version` を検証します。
5.  プラグインローカルバイナリが欠落または不一致の場合、以下を実行: `npm install --omit=dev --no-save acpx@` して再検証。

プラグイン設定でコマンド/バージョンをオーバーライドできます:

```json
{
  "plugins": {
    "entries": {
      "acpx": {
        "enabled": true,
        "config": {
          "command": "../acpx/dist/cli.js",
          "expectedVersion": "any"
        }
      }
    }
  }
}
```

注意:

-   `command` は絶対パス、相対パス、またはコマンド名 (`acpx`) を受け入れます。
-   相対パスは OpenClaw ワークスペースディレクトリから解決されます。
-   `expectedVersion: "any"` は厳密なバージョン一致を無効にします。
-   `command` がカスタムバイナリ/パスを指す場合、プラグインローカルの自動インストールは無効になります。
-   OpenClaw の起動はバックエンド健全性チェックの実行中も非ブロッキングのままです。

[プラグイン](./plugin.md)を参照してください。

## パーミッション設定

ACP セッションは非対話的に実行されます — ファイル書き込みやシェル実行の許可プロンプトを承認または拒否する TTY はありません。acpx プラグインは、パーミッションの処理方法を制御する 2 つの設定キーを提供します:

### permissionMode

ハーネスエージェントがプロンプトなしで実行できる操作を制御します。

| 値 | 動作 |
| --- | --- |
| `approve-all` | すべてのファイル書き込みとシェルコマンドを自動承認。 |
| `approve-reads` | 読み取りのみを自動承認; 書き込みと実行にはプロンプトが必要。 |
| `deny-all` | すべてのパーミッションプロンプトを拒否。 |

### nonInteractivePermissions

パーミッションプロンプトが表示されるはずだが、対話型 TTY が利用できない場合 (ACP セッションでは常に該当) の動作を制御します。

| 値 | 動作 |
| --- | --- |
| `fail` | `AcpRuntimeError` でセッションを中止。 **(デフォルト)** |
| `deny` | パーミッションを暗黙的に拒否して続行 (優雅な劣化)。 |

### 設定

プラグイン設定で設定:

```bash
openclaw config set plugins.entries.acpx.config.permissionMode approve-all
openclaw config set plugins.entries.acpx.config.nonInteractivePermissions fail
```

これらの値を変更した後はゲートウェイを再起動してください。

> **重要:** OpenClaw は現在、デフォルトで `permissionMode=approve-reads` および `nonInteractivePermissions=fail` です。非対話型 ACP セッションでは、パーミッションプロンプトをトリガーする書き込みまたは実行は `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` で失敗する可能性があります。パーミッションを制限する必要がある場合は、セッションがクラッシュするのではなく優雅に劣化するように `nonInteractivePermissions` を `deny` に設定してください。

## トラブルシューティング

| 症状 | 考えられる原因 | 修正方法 |
| --- | --- | --- |
| `ACP runtime backend is not configured` | バックエンドプラグインが欠落または無効。 | バックエンドプラグインをインストールして有効化し、`/acp doctor` を実行。 |
| `ACP is disabled by policy (acp.enabled=false)` | ACP がグローバルに無効。 | `acp.enabled=true` を設定。 |
| `ACP dispatch is disabled by policy (acp.dispatch.enabled=false)` | 通常のスレッドメッセージからのディスパッチが無効。 | `acp.dispatch.enabled=true` を設定。 |
| `ACP agent "" is not allowed by policy` | エージェントが許可リストにない。 | 許可された `agentId` を使用するか、`acp.allowedAgents` を更新。 |
| `Unable to resolve session target: ...` | 不正なキー/ID/ラベルトークン。 | `/acp sessions` を実行し、正確なキー/ラベルをコピーして再試行。 |
| `--thread here requires running /acp spawn inside an active ... thread` | `--thread here` がスレッドコンテキスト外で使用された。 | ターゲットスレッドに移動するか、`--thread auto`/`off` を使用。 |
| `Only <user-id> can rebind this thread.` | 別のユーザーがスレッドバインディングを所有。 | 所有者として再バインドするか、別のスレッドを使用。 |
| `Thread bindings are unavailable for .` | アダプターにスレッドバインディング機能がない。 | `--thread off` を使用するか、サポートされているアダプター/チャネルに移動。 |
| `Sandboxed sessions cannot spawn ACP sessions ...` | ACP ランタイムはホスト側; 要求元セッションはサンドボックス化されている。 | サンドボックス化されたセッションからは `runtime="subagent"` を使用するか、非サンドボックス化されたセッションから ACP 生成を実行。 |
| `sessions_spawn sandbox="require" is unsupported for runtime="acp" ...` | ACP ランタイムに対して `sandbox="require"` が要求された。 | 必須のサンドボックス化には `runtime="subagent"` を使用するか、非サンドボックス化されたセッションから `sandbox="inherit"` で ACP を使用。 |
| バインドされたセッションの ACP メタデータが欠落 | 古い/削除された ACP セッションメタデータ。 | `/acp spawn` で再作成し、スレッドを再バインド/フォーカス。 |
| `AcpRuntimeError: Permission prompt unavailable in non-interactive mode` | `permissionMode` が非対話型 ACP セッションでの書き込み/実行をブロック。 | `plugins.entries.acpx.config.permissionMode` を `approve-all` に設定し、ゲートウェイを再起動。[パーミッション設定](#permission-configuration)を参照。 |
| ACP セッションが出力が少なく早期に失敗 | パーミッションプロンプトが `permissionMode`/`nonInteractivePermissions` によってブロックされている。 | ゲートウェイログで `AcpRuntimeError` を確認。完全なパーミッションには `permissionMode=approve-all` を設定; 優雅な劣化には `nonInteractivePermissions=deny` を設定。 |
| ACP セッションが作業完了後も無期限に停止 | ハーネスプロセスは終了したが、ACP セッションが完了を報告しなかった。 | `ps aux \| grep acpx` で監視; 古いプロセスを手動で強制終了。 |

[サブエージェント](./subagents.md)[マルチエージェントサンドボックス & ツール](./multi-agent-sandbox-tools.md)