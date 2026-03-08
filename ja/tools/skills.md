

  スキル

  
# スキル

OpenClaw は、エージェントにツールの使用方法を教えるために **[AgentSkills](https://agentskills.io) 互換**のスキルフォルダを使用します。各スキルは、YAML フロントマターと説明を含む `SKILL.md` を持つディレクトリです。OpenClaw は **バンドル済みスキル** に加えてオプションのローカルオーバーライドを読み込み、環境、設定、バイナリの存在に基づいてロード時にフィルタリングします。

## ロケーションと優先順位

スキルは **3つの場所** から読み込まれます:

1.  **バンドル済みスキル**: インストールに同梱 (npm パッケージまたは OpenClaw.app)
2.  **管理/ローカルスキル**: `~/.openclaw/skills`
3.  **ワークスペーススキル**: `/skills`

スキル名が競合する場合、優先順位は: `/skills` (最高) → `~/.openclaw/skills` → バンドル済みスキル (最低) となります。さらに、`~/.openclaw/openclaw.json` の `skills.load.extraDirs` を介して追加のスキルフォルダ (最低優先順位) を設定できます。

## エージェントごと vs 共有スキル

**マルチエージェント** セットアップでは、各エージェントは独自のワークスペースを持ちます。つまり:

-   **エージェントごとのスキル** は、そのエージェント専用の `/skills` に配置されます。
-   **共有スキル** は `~/.openclaw/skills` (管理/ローカル) に配置され、同じマシン上の **すべてのエージェント** から見えます。
-   **共有フォルダ** は、複数のエージェントで使用される共通のスキルパックを希望する場合、`skills.load.extraDirs` (最低優先順位) を介して追加することもできます。

同じスキル名が複数の場所に存在する場合、通常の優先順位が適用されます: ワークスペースが優先され、次に管理/ローカル、次にバンドル済みとなります。

## プラグイン + スキル

プラグインは、`openclaw.plugin.json` に `skills` ディレクトリをリストすることで (プラグインルートからの相対パス)、独自のスキルを同梱できます。プラグインスキルは、プラグインが有効になると読み込まれ、通常のスキル優先順位ルールに参加します。プラグインの設定エントリで `metadata.openclaw.requires.config` を介してゲーティングできます。発見/設定については [プラグイン](./plugin.md) を、それらのスキルが教えるツールサーフェスについては [ツール](../tools.md) を参照してください。

## ClawHub (インストール + 同期)

ClawHub は OpenClaw の公開スキルレジストリです。[https://clawhub.com](https://clawhub.com) で閲覧できます。スキルの発見、インストール、更新、バックアップに使用します。完全ガイド: [ClawHub](./clawhub.md)。一般的なフロー:

-   ワークスペースにスキルをインストール:
    -   `clawhub install <skill-slug>`
-   インストール済みのすべてのスキルを更新:
    -   `clawhub update --all`
-   同期 (スキャン + 更新を公開):
    -   `clawhub sync --all`

デフォルトでは、`clawhub` は現在の作業ディレクトリの下の `./skills` にインストールします (または設定された OpenClaw ワークスペースにフォールバックします)。OpenClaw は次のセッションでそれを `/skills` として取得します。

## セキュリティに関する注意

-   サードパーティ製スキルは **信頼できないコード** として扱ってください。有効にする前に内容を確認してください。
-   信頼できない入力やリスクの高いツールにはサンドボックス化された実行を優先してください。[サンドボックス化](../gateway/sandboxing.md) を参照してください。
-   ワークスペースおよび追加ディレクトリのスキル発見は、解決された実パスが設定されたルート内に留まるスキルルートと `SKILL.md` ファイルのみを受け入れます。
-   `skills.entries.*.env` と `skills.entries.*.apiKey` は、そのエージェントターンの **ホスト** プロセスにシークレットを注入します (サンドボックスではありません)。シークレットをプロンプトやログから遠ざけてください。
-   より広範な脅威モデルとチェックリストについては、[セキュリティ](../gateway/security.md) を参照してください。

## フォーマット (AgentSkills + Pi 互換)

`SKILL.md` には少なくとも以下を含める必要があります:

```
---
name: nano-banana-pro
description: Gemini 3 Pro Image を介して画像を生成または編集する
---
```

注意:

-   レイアウト/意図については AgentSkills 仕様に従います。
-   組み込みエージェントが使用するパーサーは、**単一行** のフロントマターキーのみをサポートします。
-   `metadata` は **単一行の JSON オブジェクト** である必要があります。
-   説明内でスキルフォルダパスを参照するには `{baseDir}` を使用します。
-   オプションのフロントマターキー:
    -   `homepage` — macOS スキル UI で「ウェブサイト」として表示される URL (`metadata.openclaw.homepage` でもサポート)。
    -   `user-invocable` — `true|false` (デフォルト: `true`)。`true` の場合、スキルはユーザースラッシュコマンドとして公開されます。
    -   `disable-model-invocation` — `true|false` (デフォルト: `false`)。`true` の場合、スキルはモデルプロンプトから除外されます (ユーザー呼び出しでは引き続き利用可能)。
    -   `command-dispatch` — `tool` (オプション)。`tool` に設定すると、スラッシュコマンドはモデルをバイパスし、直接ツールにディスパッチします。
    -   `command-tool` — `command-dispatch: tool` が設定されているときに呼び出すツール名。
    -   `command-arg-mode` — `raw` (デフォルト)。ツールディスパッチの場合、生の引数文字列をツールに転送します (コアパースなし)。ツールは次のパラメータで呼び出されます: `{ command: "", commandName: "", skillName: "" }`。

## ゲーティング (ロード時フィルター)

OpenClaw は `metadata` (単一行 JSON) を使用して **ロード時にスキルをフィルタリング** します:

```
---
name: nano-banana-pro
description: Gemini 3 Pro Image を介して画像を生成または編集する
metadata:
  {
    "openclaw":
      {
        "requires": { "bins": ["uv"], "env": ["GEMINI_API_KEY"], "config": ["browser.enabled"] },
        "primaryEnv": "GEMINI_API_KEY",
      },
  }
---
```

`metadata.openclaw` の下のフィールド:

-   `always: true` — 常にスキルを含める (他のゲートをスキップ)。
-   `emoji` — macOS スキル UI で使用されるオプションの絵文字。
-   `homepage` — macOS スキル UI で「ウェブサイト」として表示されるオプションの URL。
-   `os` — プラットフォームのオプションリスト (`darwin`, `linux`, `win32`)。設定されている場合、スキルはそれらの OS でのみ有効です。
-   `requires.bins` — リスト; 各バイナリは `PATH` 上に存在する必要があります。
-   `requires.anyBins` — リスト; 少なくとも 1 つは `PATH` 上に存在する必要があります。
-   `requires.env` — リスト; 環境変数が存在する **または** 設定で提供されている必要があります。
-   `requires.config` — 真実値でなければならない `openclaw.json` パスのリスト。
-   `primaryEnv` — `skills.entries..apiKey` に関連付けられた環境変数名。
-   `install` — macOS スキル UI で使用されるインストーラ仕様のオプション配列 (brew/node/go/uv/download)。

サンドボックス化に関する注意:

-   `requires.bins` はスキルロード時に **ホスト** でチェックされます。
-   エージェントがサンドボックス化されている場合、バイナリは **コンテナ内** にも存在する必要があります。`agents.defaults.sandbox.docker.setupCommand` (またはカスタムイメージ) を介してインストールしてください。`setupCommand` はコンテナ作成後に一度実行されます。パッケージインストールには、ネットワークエグレス、書き込み可能なルート FS、およびサンドボックス内の root ユーザーも必要です。例: `summarize` スキル (`skills/summarize/SKILL.md`) は、そこで実行するためにサンドボックスコンテナ内に `summarize` CLI を必要とします。

インストーラの例:

```
---
name: gemini
description: Gemini CLI を使用してコーディング支援と Google 検索ルックアップを行う。
metadata:
  {
    "openclaw":
      {
        "emoji": "♊️",
        "requires": { "bins": ["gemini"] },
        "install":
          [
            {
              "id": "brew",
              "kind": "brew",
              "formula": "gemini-cli",
              "bins": ["gemini"],
              "label": "Gemini CLI をインストール (brew)",
            },
          ],
      },
  }
---
```

注意:

-   複数のインストーラがリストされている場合、ゲートウェイは **単一の** 優先オプションを選択します (利用可能な場合は brew、それ以外は node)。
-   すべてのインストーラが `download` の場合、OpenClaw は各エントリをリストして利用可能なアーティファクトを確認できるようにします。
-   インストーラ仕様には、プラットフォームごとにオプションをフィルタリングするための `os: ["darwin"|"linux"|"win32"]` を含めることができます。
-   Node インストールは `openclaw.json` の `skills.install.nodeManager` を尊重します (デフォルト: npm; オプション: npm/pnpm/yarn/bun)。これは **スキルインストール** のみに影響します。ゲートウェイランタイムは引き続き Node であるべきです (Bun は WhatsApp/Telegram には推奨されません)。
-   Go インストール: `go` がなく `brew` が利用可能な場合、ゲートウェイはまず Homebrew 経由で Go をインストールし、可能な場合は `GOBIN` を Homebrew の `bin` に設定します。
-   ダウンロードインストール: `url` (必須), `archive` (`tar.gz` | `tar.bz2` | `zip`), `extract` (デフォルト: アーカイブ検出時に自動), `stripComponents`, `targetDir` (デフォルト: `~/.openclaw/tools/`)。

`metadata.openclaw` が存在しない場合、スキルは常に有効です (設定で無効にされている場合、またはバンドル済みスキルに対して `skills.allowBundled` でブロックされている場合を除く)。

## 設定オーバーライド (~/.openclaw/openclaw.json)

バンドル済み/管理スキルは、有効/無効を切り替えたり、環境変数値を提供したりできます:

```json
{
  skills: {
    entries: {
      "nano-banana-pro": {
        enabled: true,
        apiKey: { source: "env", provider: "default", id: "GEMINI_API_KEY" }, // またはプレーンテキスト文字列
        env: {
          GEMINI_API_KEY: "GEMINI_KEY_HERE",
        },
        config: {
          endpoint: "https://example.invalid",
          model: "nano-pro",
        },
      },
      peekaboo: { enabled: true },
      sag: { enabled: false },
    },
  },
}
```

注意: スキル名にハイフンが含まれる場合、キーを引用符で囲んでください (JSON5 は引用符付きキーを許可します)。設定キーはデフォルトで **スキル名** と一致します。スキルが `metadata.openclaw.skillKey` を定義している場合、`skills.entries` の下でそのキーを使用してください。ルール:

-   `enabled: false` は、スキルがバンドル済み/インストール済みであっても無効にします。
-   `env`: 変数がプロセス内ですでに設定されていない **場合にのみ** 注入されます。
-   `apiKey`: `metadata.openclaw.primaryEnv` を宣言するスキルのための便利機能。プレーンテキスト文字列または SecretRef オブジェクト (`{ source, provider, id }`) をサポートします。
-   `config`: カスタムのスキルごとのフィールドのためのオプションバッグ。カスタムキーはここに配置する必要があります。
-   `allowBundled`: **バンドル済み** スキルのみのためのオプションの許可リスト。設定されている場合、リスト内のバンドル済みスキルのみが有効です (管理/ワークスペーススキルは影響を受けません)。

## 環境変数注入 (エージェント実行ごと)

エージェント実行が開始されると、OpenClaw は以下を行います:

1.  スキルメタデータを読み取ります。
2.  `skills.entries..env` または `skills.entries..apiKey` を `process.env` に適用します。
3.  **有効な** スキルを含むシステムプロンプトを構築します。
4.  実行終了後に元の環境を復元します。

これは **エージェント実行にスコープ化** されており、グローバルなシェル環境ではありません。

## セッションスナップショット (パフォーマンス)

OpenClaw は、**セッション開始時** に有効なスキルのスナップショットを取得し、同じセッション内の後続のターンでそのリストを再利用します。スキルや設定への変更は、次の新しいセッションで有効になります。スキルは、スキルウォッチャーが有効になっている場合、または新しい有効なリモートノードが出現した場合 (下記参照) に、セッション中にも更新できます。これは **ホットリロード** と考えてください: 更新されたリストは次のエージェントターンで取得されます。

## リモート macOS ノード (Linux ゲートウェイ)

ゲートウェイが Linux で実行されているが、**macOS ノード** が **`system.run` 許可あり** で接続されている場合 (Exec 承認セキュリティが `deny` に設定されていない)、OpenClaw は必要なバイナリがそのノード上に存在するときに macOS 専用スキルを有効として扱うことができます。エージェントは `nodes` ツール (通常は `nodes.run`) を介してそれらのスキルを実行するべきです。これは、ノードがそのコマンドサポートを報告し、`system.run` を介したバイナリプローブに依存します。macOS ノードが後でオフラインになっても、スキルは表示されたままです。ノードが再接続するまで、呼び出しは失敗する可能性があります。

## スキルウォッチャー (自動更新)

デフォルトでは、OpenClaw はスキルフォルダを監視し、`SKILL.md` ファイルが変更されたときにスキルスナップショットを更新します。`skills.load` の下で設定します:

```json
{
  skills: {
    load: {
      watch: true,
      watchDebounceMs: 250,
    },
  },
}
```

## トークンへの影響 (スキルリスト)

スキルが有効な場合、OpenClaw は利用可能なスキルのコンパクトな XML リストをシステムプロンプトに注入します (`pi-coding-agent` の `formatSkillsForPrompt` 経由)。コストは決定的です:

-   **ベースオーバーヘッド (スキルが ≥1 の場合のみ):** 195 文字。
-   **スキルごと:** 97 文字 + XML エスケープされた ``, ``, `` 値の長さ。

計算式 (文字数):

```bash
total = 195 + Σ (97 + len(name_escaped) + len(description_escaped) + len(location_escaped))
```

注意:

-   XML エスケープは `& < > " '` をエンティティ (`&amp;`, `<` など) に展開し、長さを増加させます。
-   トークン数はモデルトークナイザーによって異なります。大まかな OpenAI スタイルの見積もりは ~4 文字/トークンなので、**97 文字 ≈ 24 トークン** に実際のフィールド長を加えたものがスキルごとのコストです。

## 管理スキルのライフサイクル

OpenClaw は、インストールの一部として **バンドル済みスキル** のベースラインセットを同梱します (npm パッケージまたは OpenClaw.app)。`~/.openclaw/skills` はローカルオーバーライド用に存在します (例えば、バンドル済みコピーを変更せずにスキルをピン留め/パッチ適用するため)。ワークスペーススキルはユーザー所有であり、名前が競合した場合には両方をオーバーライドします。

## 設定リファレンス

完全な設定スキーマについては [スキル設定](./skills-config.md) を参照してください。

## さらに多くのスキルをお探しですか?

[https://clawhub.com](https://clawhub.com) を閲覧してください。

* * *

[スラッシュコマンド](./slash-commands.md)[スキル設定](./skills-config.md)