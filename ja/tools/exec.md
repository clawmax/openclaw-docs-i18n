

  組み込みツール

  
# Exec ツール

ワークスペース内でシェルコマンドを実行します。`process` を介したフォアグラウンドおよびバックグラウンド実行をサポートします。`process` が許可されていない場合、`exec` は同期的に実行され、`yieldMs`/`background` は無視されます。バックグラウンドセッションはエージェントごとにスコープされます。`process` は同じエージェントからのセッションのみを認識します。

## パラメータ

-   `command` (必須)
-   `workdir` (デフォルトは cwd)
-   `env` (キー/値のオーバーライド)
-   `yieldMs` (デフォルト 10000): 遅延後に自動バックグラウンド化
-   `background` (bool): 即時バックグラウンド化
-   `timeout` (秒, デフォルト 1800): 期限切れで強制終了
-   `pty` (bool): 利用可能な場合、擬似端末で実行 (TTY専用CLI、コーディングエージェント、ターミナルUI)
-   `host` (`sandbox | gateway | node`): 実行場所
-   `security` (`deny | allowlist | full`): `gateway`/`node` のための強制モード
-   `ask` (`off | on-miss | always`): `gateway`/`node` のための承認プロンプト
-   `node` (string): `host=node` のためのノードID/名前
-   `elevated` (bool): 昇格モードを要求 (ゲートウェイホスト); `elevated` が `full` に解決された場合のみ `security=full` が強制されます

注記:

-   `host` のデフォルトは `sandbox` です。
-   `elevated` は、サンドボックスがオフの場合 (exec は既にホスト上で実行されている) は無視されます。
-   `gateway`/`node` の承認は `~/.openclaw/exec-approvals.json` によって制御されます。
-   `node` には、ペアリングされたノード (コンパニオンアプリまたはヘッドレスノードホスト) が必要です。
-   複数のノードが利用可能な場合、`exec.node` または `tools.exec.node` を設定して選択します。
-   非Windowsホストでは、exec は `SHELL` が設定されている場合それを使用します。`SHELL` が `fish` の場合、fish非互換スクリプトを避けるため、`PATH` から `bash` (または `sh`) を優先し、どちらも存在しない場合は `SHELL` にフォールバックします。
-   Windowsホストでは、exec は PowerShell 7 (`pwsh`) の検出 (Program Files, ProgramW6432, 次に PATH) を優先し、次に Windows PowerShell 5.1 にフォールバックします。
-   ホスト実行 (`gateway`/`node`) は、バイナリハイジャックやコード注入を防ぐため、`env.PATH` とローダーオーバーライド (`LD_*`/`DYLD_*`) を拒否します。
-   OpenClaw は、spawnされたコマンド環境 (PTYおよびサンドボックス実行を含む) に `OPENCLAW_SHELL=exec` を設定し、シェル/プロファイルルールが exec-tool コンテキストを検出できるようにします。
-   重要: サンドボックスは**デフォルトでオフ**です。サンドボックスがオフで `host=sandbox` が明示的に設定/要求された場合、exec はゲートウェイホスト上で暗黙的に実行される代わりに、閉じて失敗するようになりました。サンドボックスを有効にするか、承認付きで `host=gateway` を使用してください。
-   スクリプトの事前チェック (一般的なPython/Nodeシェル構文の誤りのため) は、実効的な `workdir` 境界内のファイルのみを検査します。スクリプトパスが `workdir` の外に解決される場合、そのファイルの事前チェックはスキップされます。

## 設定

-   `tools.exec.notifyOnExit` (デフォルト: true): trueの場合、バックグラウンド化された exec セッションは終了時にシステムイベントをエンキューし、ハートビートを要求します。
-   `tools.exec.approvalRunningNoticeMs` (デフォルト: 10000): 承認ゲートされた exec がこの時間より長く実行された場合、単一の「実行中」通知を発行します (0 で無効化)。
-   `tools.exec.host` (デフォルト: `sandbox`)
-   `tools.exec.security` (デフォルト: サンドボックスの場合は `deny`、未設定時の gateway + node の場合は `allowlist`)
-   `tools.exec.ask` (デフォルト: `on-miss`)
-   `tools.exec.node` (デフォルト: 未設定)
-   `tools.exec.pathPrepend`: exec 実行 (gateway + サンドボックスのみ) のために `PATH` の先頭に追加するディレクトリのリスト。
-   `tools.exec.safeBins`: 明示的な許可リストエントリなしで実行できる、stdin専用の安全なバイナリ。動作の詳細は [安全なバイナリ](./exec-approvals.md#safe-bins-stdin-only) を参照してください。
-   `tools.exec.safeBinTrustedDirs`: `safeBins` パスチェックのために追加で明示的に信頼されるディレクトリ。`PATH` エントリは自動的に信頼されることはありません。組み込みのデフォルトは `/bin` と `/usr/bin` です。
-   `tools.exec.safeBinProfiles`: 安全なバイナリごとのオプションのカスタム argv ポリシー (`minPositional`, `maxPositional`, `allowedValueFlags`, `deniedFlags`)。

例:

```json
{
  tools: {
    exec: {
      pathPrepend: ["~/bin", "/opt/oss/bin"],
    },
  },
}
```

### PATH の取り扱い

-   `host=gateway`: ログインシェルの `PATH` を exec 環境にマージします。ホスト実行のための `env.PATH` オーバーライドは拒否されます。デーモン自体は依然として最小限の `PATH` で実行されます:
    -   macOS: `/opt/homebrew/bin`, `/usr/local/bin`, `/usr/bin`, `/bin`
    -   Linux: `/usr/local/bin`, `/usr/bin`, `/bin`
-   `host=sandbox`: コンテナ内で `sh -lc` (ログインシェル) を実行するため、`/etc/profile` が `PATH` をリセットする可能性があります。OpenClaw は、プロファイルソース取得後、内部環境変数を介して `env.PATH` を先頭に追加します (シェル補間なし)。`tools.exec.pathPrepend` もここで適用されます。
-   `host=node`: 渡されたブロックされていない env オーバーライドのみがノードに送信されます。ホスト実行のための `env.PATH` オーバーライドは拒否され、ノードホストによって無視されます。ノード上で追加の PATH エントリが必要な場合は、ノードホストサービスの環境 (systemd/launchd) を設定するか、標準的な場所にツールをインストールしてください。

エージェントごとのノードバインド (設定内のエージェントリストインデックスを使用):

```bash
openclaw config get agents.list
openclaw config set agents.list[0].tools.exec.node "node-id-or-name"
```

コントロールUI: ノードタブには、同じ設定のための小さな「Exec ノードバインド」パネルが含まれています。

## セッションオーバーライド (/exec)

`/exec` を使用して、`host`、`security`、`ask`、`node` の**セッションごとの**デフォルトを設定します。引数なしで `/exec` を送信すると、現在の値を表示します。例:

```bash
/exec host=gateway security=allowlist ask=on-miss node=mac-1
```

## 認可モデル

`/exec` は**認可された送信者** (チャネル許可リスト/ペアリング および `commands.useAccessGroups`) に対してのみ有効です。これは**セッション状態のみ**を更新し、設定を書き込みません。exec を完全に無効化するには、ツールポリシー (`tools.deny: ["exec"]` または エージェントごと) を介して拒否してください。明示的に `security=full` と `ask=off` を設定しない限り、ホスト承認は依然として適用されます。

## Exec 承認 (コンパニオンアプリ / ノードホスト)

サンドボックス化されたエージェントは、`exec` がゲートウェイまたはノードホスト上で実行される前に、リクエストごとの承認を必要とする場合があります。ポリシー、許可リスト、および UI フローについては [Exec 承認](./exec-approvals.md) を参照してください。承認が必要な場合、exec ツールは `status: "approval-pending"` と承認 id ですぐに返ります。承認 (または拒否/タイムアウト) されると、Gateway はシステムイベント (`Exec finished` / `Exec denied`) を発行します。コマンドが `tools.exec.approvalRunningNoticeMs` 後も実行中の場合は、単一の `Exec running` 通知が発行されます。

## 許可リスト + 安全なバイナリ

手動許可リスト強制は、**解決されたバイナリパスのみ**と一致します (ベース名の一致はありません)。`security=allowlist` の場合、シェルコマンドは、パイプラインのすべてのセグメントが許可リストされているか安全なバイナリである場合にのみ自動許可されます。チェイニング (`;`, `&&`, `||`) とリダイレクションは、すべてのトップレベルセグメントが許可リスト (安全なバイナリを含む) を満たさない限り、許可リストモードでは拒否されます。リダイレクションは依然としてサポートされていません。`autoAllowSkills` は、exec 承認における別の便利なパスです。これは手動パス許可リストエントリと同じではありません。厳密な明示的な信頼のためには、`autoAllowSkills` を無効にしてください。2つのコントロールを異なる目的で使用してください:

-   `tools.exec.safeBins`: 小さな、stdin専用のストリームフィルタ。
-   `tools.exec.safeBinTrustedDirs`: 安全なバイナリ実行可能パスのための明示的な追加信頼ディレクトリ。
-   `tools.exec.safeBinProfiles`: カスタム安全なバイナリのための明示的な argv ポリシー。
-   許可リスト: 実行可能パスの明示的な信頼。

`safeBins` を一般的な許可リストとして扱わず、インタープリタ/ランタイムバイナリ (例: `python3`, `node`, `ruby`, `bash`) を追加しないでください。それらが必要な場合は、明示的な許可リストエントリを使用し、承認プロンプトを有効にしたままにしてください。`openclaw security audit` は、インタープリタ/ランタイム `safeBins` エントリに明示的なプロファイルが欠けている場合に警告し、`openclaw doctor --fix` は欠けているカスタム `safeBinProfiles` エントリを足場構築できます。完全なポリシーの詳細と例については、[Exec 承認](./exec-approvals.md#safe-bins-stdin-only) および [安全なバイナリ vs 許可リスト](./exec-approvals.md#safe-bins-versus-allowlist) を参照してください。

## 例

フォアグラウンド:

```json
{ "tool": "exec", "command": "ls -la" }
```

バックグラウンド + ポーリング:

```json
{"tool":"exec","command":"npm run build","yieldMs":1000}
{"tool":"process","action":"poll","sessionId":"<id>"}
```

キー送信 (tmuxスタイル):

```json
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Enter"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["C-c"]}
{"tool":"process","action":"send-keys","sessionId":"<id>","keys":["Up","Up","Enter"]}
```

送信 (CRのみ送信):

```json
{ "tool": "process", "action": "submit", "sessionId": "<id>" }
```

貼り付け (デフォルトでブラケット付き):

```json
{ "tool": "process", "action": "paste", "sessionId": "<id>", "text": "line1\nline2\n" }
```

## apply\_patch (実験的)

`apply_patch` は、構造化されたマルチファイル編集のための `exec` のサブツールです。明示的に有効にしてください:

```json
{
  tools: {
    exec: {
      applyPatch: { enabled: true, workspaceOnly: true, allowModels: ["gpt-5.2"] },
    },
  },
}
```

注記:

-   OpenAI/OpenAI Codex モデルでのみ利用可能です。
-   ツールポリシーは依然として適用されます。`allow: ["exec"]` は暗黙的に `apply_patch` を許可します。
-   設定は `tools.exec.applyPatch` の下にあります。
-   `tools.exec.applyPatch.workspaceOnly` はデフォルトで `true` (ワークスペース内に限定) です。`apply_patch` にワークスペースディレクトリ外への書き込み/削除を意図的に許可したい場合のみ、`false` に設定してください。

[昇格モード](./elevated.md)[Exec 承認](./exec-approvals.md)

---