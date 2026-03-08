

  セキュリティとサンドボックス

  
# セキュリティ

> \[!WARNING\] **パーソナルアシスタント信頼モデル:** このガイダンスは、ゲートウェイごとに1つの信頼できるオペレーター境界（シングルユーザー/パーソナルアシスタントモデル）を前提としています。OpenClaw は、1つのエージェント/ゲートウェイを共有する複数の敵対的ユーザーに対する**敵対的マルチテナントセキュリティ境界ではありません**。混合信頼または敵対的ユーザー操作が必要な場合は、信頼境界を分割してください（別々のゲートウェイ + 認証情報、理想的には別々のOSユーザー/ホスト）。

## まずは範囲: パーソナルアシスタントセキュリティモデル

OpenClaw のセキュリティガイダンスは、**パーソナルアシスタント** デプロイメントを前提としています: 1つの信頼できるオペレーター境界、潜在的に多数のエージェント。

-   サポートされるセキュリティ態勢: ゲートウェイごとに1ユーザー/1信頼境界（境界ごとに1OSユーザー/ホスト/VPSを推奨）。
-   サポートされないセキュリティ境界: 相互に信頼できない、または敵対的なユーザーが共有する1つのゲートウェイ/エージェント。
-   敵対的ユーザー分離が必要な場合は、信頼境界ごとに分割（別々のゲートウェイ + 認証情報、理想的には別々のOSユーザー/ホスト）。
-   複数の信頼できないユーザーが1つのツール対応エージェントにメッセージを送れる場合、それらはそのエージェントに対して同じ委任されたツール権限を共有していると見なします。

このページでは、**そのモデル内での** 強化について説明します。1つの共有ゲートウェイ上での敵対的マルチテナント分離を主張するものではありません。

## クイックチェック: openclaw セキュリティ監査

参照: [正式検証（セキュリティモデル）](../security/formal-verification.md) これを定期的に実行してください（特に設定を変更したり、ネットワーク表面を公開した後）:

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

これは一般的な落とし穴（ゲートウェイ認証情報の露出、ブラウザ制御の露出、昇格された許可リスト、ファイルシステム権限）をフラグします。OpenClaw は製品であると同時に実験でもあります: フロンティアモデルの動作を実際のメッセージング表面と実際のツールに配線しています。 **「完全に安全な」セットアップはありません。** 目標は以下について意図的であることです:

-   誰がボットと話せるか
-   ボットがどこで行動を許可されるか
-   ボットが何に触れられるか

動作する最小限のアクセスから始め、自信がつくにつれて広げてください。

## デプロイメントの前提条件（重要）

OpenClaw は、ホストと設定境界が信頼されていることを前提としています:

-   誰かがゲートウェイホストの状態/設定（`~/.openclaw`、`openclaw.json` を含む）を変更できる場合、その人を信頼できるオペレーターとして扱います。
-   相互に信頼できない/敵対的な複数のオペレーターのために1つのゲートウェイを実行することは、**推奨されないセットアップ**です。
-   混合信頼チームの場合は、別々のゲートウェイ（または少なくとも別々のOSユーザー/ホスト）で信頼境界を分割してください。
-   OpenClaw は1台のマシン上で複数のゲートウェイインスタンスを実行できますが、推奨される運用ではクリーンな信頼境界の分離を優先します。
-   推奨されるデフォルト: マシン/ホスト（またはVPS）ごとに1ユーザー、そのユーザーのための1ゲートウェイ、およびそのゲートウェイ内の1つ以上のエージェント。
-   複数のユーザーが OpenClaw を必要とする場合は、ユーザーごとに1つのVPS/ホストを使用してください。

### 実用的な帰結（オペレーター信頼境界）

1つのゲートウェイインスタンス内では、認証されたオペレーターアクセスは、ユーザーごとのテナントロールではなく、信頼されたコントロールプレーンロールです。

-   読み取り/コントロールプレーンアクセスを持つオペレーターは、設計上、ゲートウェイセッションメタデータ/履歴を検査できます。
-   セッション識別子（`sessionKey`、セッションID、ラベル）は、認証トークンではなく、ルーティングセレクターです。
-   例: `sessions.list`、`sessions.preview`、`chat.history` などのメソッドに対するオペレーターごとの分離を期待することは、このモデルの範囲外です。
-   敵対的ユーザー分離が必要な場合は、信頼境界ごとに別々のゲートウェイを実行してください。
-   1台のマシン上での複数ゲートウェイは技術的に可能ですが、マルチユーザー分離のための推奨されるベースラインではありません。

## パーソナルアシスタントモデル（マルチテナントバスではない）

OpenClaw は、パーソナルアシスタントセキュリティモデルとして設計されています: 1つの信頼できるオペレーター境界、潜在的に多数のエージェント。

-   複数の人が1つのツール対応エージェントにメッセージを送れる場合、それぞれが同じ権限セットを操縦できます。
-   ユーザーごとのセッション/メモリ分離はプライバシーに役立ちますが、共有エージェントをユーザーごとのホスト認可に変換するものではありません。
-   ユーザーが互いに敵対的である可能性がある場合は、信頼境界ごとに別々のゲートウェイ（または別々のOSユーザー/ホスト）を実行してください。

### 共有 Slack ワークスペース: 実際のリスク

「Slack 内の誰もがボットにメッセージを送れる」場合、中核的なリスクは委任されたツール権限です:

-   許可された送信者は誰でも、エージェントのポリシー内でツール呼び出し（`exec`、ブラウザ、ネットワーク/ファイルツール）を誘発できます;
-   1人の送信者からのプロンプト/コンテンツインジェクションは、共有状態、デバイス、または出力に影響を与えるアクションを引き起こす可能性があります;
-   1つの共有エージェントが機密性の高い認証情報/ファイルを持っている場合、許可された送信者は誰でも、ツール使用を介した流出を潜在的に駆動できます。

チームワークフローには最小限のツールを持つ別々のエージェント/ゲートウェイを使用し、個人データエージェントはプライベートに保ってください。

### 会社共有エージェント: 許容可能なパターン

これは、そのエージェントを使用する全員が同じ信頼境界内にあり（例えば1つの会社チーム）、エージェントが厳密に業務範囲に限定されている場合に許容可能です。

-   専用のマシン/VM/コンテナで実行します;
-   そのランタイムには専用のOSユーザー + 専用のブラウザ/プロファイル/アカウントを使用します;
-   そのランタイムに個人のApple/Googleアカウントや個人のパスワードマネージャー/ブラウザプロファイルでサインインしないでください。

同じランタイムで個人と会社のアイデンティティを混在させると、分離が崩れ、個人データ露出リスクが高まります。

## ゲートウェイとノードの信頼概念

ゲートウェイとノードを、異なる役割を持つ1つのオペレーター信頼ドメインとして扱います:

-   **ゲートウェイ** は、コントロールプレーンとポリシー表面です（`gateway.auth`、ツールポリシー、ルーティング）。
-   **ノード** は、そのゲートウェイとペアリングされたリモート実行表面です（コマンド、デバイスアクション、ホストローカル機能）。
-   ゲートウェイに認証された呼び出し元は、ゲートウェイスコープで信頼されます。ペアリング後、ノードアクションはそのノード上の信頼されたオペレーターアクションです。
-   `sessionKey` は、ユーザーごとの認証ではなく、ルーティング/コンテキスト選択です。
-   実行承認（許可リスト + 確認）は、オペレーターの意図のためのガードレールであり、敵対的マルチテナント分離ではありません。

敵対的ユーザー分離が必要な場合は、OSユーザー/ホストごとに信頼境界を分割し、別々のゲートウェイを実行してください。

## 信頼境界マトリックス

リスクをトリアージする際のクイックモデルとしてこれを使用してください:

| 境界または制御 | 意味 | よくある誤解 |
| --- | --- | --- |
| `gateway.auth` (トークン/パスワード/デバイス認証) | ゲートウェイAPIへの呼び出し元を認証します | 「すべてのフレームでメッセージごとの署名が必要でなければ安全ではない」 |
| `sessionKey` | コンテキスト/セッション選択のためのルーティングキー | 「セッションキーはユーザー認証境界である」 |
| プロンプト/コンテンツガードレール | モデル悪用リスクを低減します | 「プロンプトインジェクションだけでは認証バイパスが証明される」 |
| `canvas.eval` / ブラウザ評価 | 有効にした場合の意図的なオペレーター機能 | 「この信頼モデルでは、任意のJS evalプリミティブは自動的に脆弱性である」 |
| ローカルTUI `!` シェル | 明示的なオペレーター起動のローカル実行 | 「ローカルシェル便利コマンドはリモートインジェクションである」 |
| ノードペアリングとノードコマンド | ペアリングされたデバイス上のオペレーターレベルのリモート実行 | 「リモートデバイス制御はデフォルトで信頼できないユーザーアクセスとして扱われるべきである」 |

## 設計上脆弱性ではないもの

これらのパターンはよく報告され、実際の境界バイパスが示されない限り、通常はアクションなしとしてクローズされます:

-   ポリシー/認証/サンドボックスバイパスなしのプロンプトインジェクションのみの連鎖。
-   1つの共有ホスト/設定での敵対的マルチテナント操作を前提とする主張。
-   共有ゲートウェイセットアップで、通常のオペレーター読み取りパスアクセス（例: `sessions.list`/`sessions.preview`/`chat.history`）をIDORとして分類する主張。
-   ローカルホストのみのデプロイメントの所見（例: ループバック専用ゲートウェイでのHSTS）。
-   このリポジトリに存在しないインバウンドパスに対するDiscordインバウンドWebhook署名の所見。
-   `sessionKey` を認証トークンとして扱う「ユーザーごとの認可が欠如している」という所見。

## 研究者向け事前チェックリスト

GHSAを開く前に、以下のすべてを確認してください:

1.  再現が最新の `main` または最新リリースでも機能する。
2.  レポートには正確なコードパス（`file`、関数、行範囲）とテスト済みバージョン/コミットが含まれている。
3.  影響が文書化された信頼境界を越えている（単なるプロンプトインジェクションではない）。
4.  主張が [対象外](https://github.com/openclaw/openclaw/blob/main/SECURITY.md#out-of-scope) にリストされていない。
5.  既存のアドバイザリが重複していないか確認済み（該当する場合は正規のGHSAを再利用）。
6.  デプロイメントの前提条件が明示的である（ループバック/ローカル vs 公開、信頼できる vs 信頼できないオペレーター）。

## 60秒での強化ベースライン

まずこのベースラインを使用し、信頼できるエージェントごとにツールを選択的に再有効化してください:

```json
{
  gateway: {
    mode: "local",
    bind: "loopback",
    auth: { mode: "token", token: "replace-with-long-random-token" },
  },
  session: {
    dmScope: "per-channel-peer",
  },
  tools: {
    profile: "messaging",
    deny: ["group:automation", "group:runtime", "group:fs", "sessions_spawn", "sessions_send"],
    fs: { workspaceOnly: true },
    exec: { security: "deny", ask: "always" },
    elevated: { enabled: false },
  },
  channels: {
    whatsapp: { dmPolicy: "pairing", groups: { "*": { requireMention: true } } },
  },
}
```

これにより、ゲートウェイはローカルのみに保たれ、DMが分離され、コントロールプレーン/ランタイムツールがデフォルトで無効になります。

## 共有受信トレイのクイックルール

複数の人があなたのボットにDMを送れる場合:

-   `session.dmScope: "per-channel-peer"`（またはマルチアカウントチャネルの場合は `"per-account-channel-peer"`）を設定します。
-   `dmPolicy: "pairing"` または厳格な許可リストを維持します。
-   共有DMと広範なツールアクセスを組み合わせないでください。
-   これは協調的/共有受信トレイを強化しますが、ユーザーがホスト/設定書き込みアクセスを共有する場合の敵対的共同テナント分離として設計されていません。

### 監査がチェックするもの（概要）

-   **インバウンドアクセス** (DMポリシー、グループポリシー、許可リスト): 見知らぬ人がボットをトリガーできるか？
-   **ツールの爆発半径** (昇格ツール + オープンルーム): プロンプトインジェクションがシェル/ファイル/ネットワークアクションに変わる可能性はあるか？
-   **ネットワーク露出** (ゲートウェイバインド/認証、Tailscale Serve/Funnel、弱い/短い認証トークン)。
-   **ブラウザ制御露出** (リモートノード、リレーポート、リモートCDPエンドポイント)。
-   **ローカルディスクの衛生状態** (権限、シンボリックリンク、設定インクルード、「同期フォルダ」パス)。
-   **プラグイン** (明示的な許可リストなしに拡張機能が存在する)。
-   **ポリシードリフト/設定ミス** (サンドボックスDocker設定が構成されているがサンドボックスモードがオフ; マッチングが正確なコマンド名のみ（例: `system.run`）でシェルテキストを検査しないため効果のない `gateway.nodes.denyCommands` パターン; 危険な `gateway.nodes.allowCommands` エントリ; グローバル `tools.profile="minimal"` がエージェントごとのプロファイルで上書きされる; 拡張プラグインツールが寛容なツールポリシー下で到達可能)。
-   **ランタイム期待値のドリフト** (例: サンドボックスモードがオフの状態での `tools.exec.host="sandbox"` は、ゲートウェイホスト上で直接実行される)。
-   **モデルの衛生状態** (設定されたモデルがレガシーに見える場合に警告; ハードブロックではない)。

`--deep` を実行すると、OpenClaw はベストエフォートでライブゲートウェイプローブも試みます。

## 認証情報ストレージマップ

アクセスを監査したり、何をバックアップするか決定する際にこれを使用してください:

-   **WhatsApp**: `~/.openclaw/credentials/whatsapp//creds.json`
-   **Telegram ボットトークン**: 設定/環境変数 または `channels.telegram.tokenFile`
-   **Discord ボットトークン**: 設定/環境変数 または SecretRef (env/file/exec プロバイダー)
-   **Slack トークン**: 設定/環境変数 (`channels.slack.*`)
-   **ペアリング許可リスト**:
    -   `~/.openclaw/credentials/-allowFrom.json` (デフォルトアカウント)
    -   `~/.openclaw/credentials/--allowFrom.json` (非デフォルトアカウント)
-   **モデル認証プロファイル**: `~/.openclaw/agents//agent/auth-profiles.json`
-   **ファイルバックアップシークレットペイロード（オプション）**: `~/.openclaw/secrets.json`
-   **レガシーOAuthインポート**: `~/.openclaw/credentials/oauth.json`

## セキュリティ監査チェックリスト

監査が所見を出力したら、これを優先順位として扱ってください:

1.  **「オープン」+ ツール有効化のもの**: まずDM/グループをロックダウン（ペアリング/許可リスト）、次にツールポリシー/サンドボックスを厳格化。
2.  **パブリックネットワーク露出** (LANバインド、Funnel、認証欠如): 直ちに修正。
3.  **ブラウザ制御リモート露出**: オペレーターアクセスのように扱う（tailnet内のみ、ノードは意図的にペアリング、公開露出は避ける）。
4.  **権限**: 状態/設定/認証情報/認証がグループ/ワールド読み取り可能でないことを確認。
5.  **プラグイン/拡張機能**: 明示的に信頼するもののみをロード。
6.  **モデル選択**: ツールを持つボットには、最新の命令強化モデルを優先。

## セキュリティ監査用語集

実際のデプロイメントで最もよく見られる可能性の高い高シグナル `checkId` 値（網羅的ではありません）:

| `checkId` | 重大度 | 重要性 | 主な修正キー/パス | 自動修正 |
| --- | --- | --- | --- | --- |
| `fs.state_dir.perms_world_writable` | critical | 他のユーザー/プロセスがOpenClaw状態全体を変更可能 | `~/.openclaw` のファイルシステム権限 | yes |
| `fs.config.perms_writable` | critical | 他のユーザーが認証/ツールポリシー/設定を変更可能 | `~/.openclaw/openclaw.json` のファイルシステム権限 | yes |
| `fs.config.perms_world_readable` | critical | 設定がトークン/設定を露出する可能性 | 設定ファイルのファイルシステム権限 | yes |
| `gateway.bind_no_auth` | critical | 共有秘密なしのリモートバインド | `gateway.bind`, `gateway.auth.*` | no |
| `gateway.loopback_no_auth` | critical | リバースプロキシされたループバックが認証なしになる可能性 | `gateway.auth.*`, プロキシ設定 | no |
| `gateway.http.no_auth` | warn/critical | `auth.mode="none"` で到達可能なゲートウェイHTTP API | `gateway.auth.mode`, `gateway.http.endpoints.*` | no |
| `gateway.tools_invoke_http.dangerous_allow` | warn/critical | HTTP API経由で危険なツールを再有効化 | `gateway.tools.allow` | no |
| `gateway.nodes.allow_commands_dangerous` | warn/critical | 高影響ノードコマンドを有効化 (カメラ/画面/連絡先/カレンダー/SMS) | `gateway.nodes.allowCommands` | no |
| `gateway.tailscale_funnel` | critical | パブリックインターネット露出 | `gateway.tailscale.mode` | no |
| `gateway.control_ui.allowed_origins_required` | critical | 明示的なブラウザオリジン許可リストなしの非ループバックControl UI | `gateway.controlUi.allowedOrigins` | no |
| `gateway.control_ui.host_header_origin_fallback` | warn/critical | Hostヘッダーオリジンフォールバックを有効化 (DNSリバインディング強化のダウングレード) | `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback` | no |
| `gateway.control_ui.insecure_auth` | warn | 非セキュア認証互換性トグルが有効 | `gateway.controlUi.allowInsecureAuth` | no |
| `gateway.control_ui.device_auth_disabled` | critical | デバイスアイデンティティチェックを無効化 | `gateway.controlUi.dangerouslyDisableDeviceAuth` | no |
| `gateway.real_ip_fallback_enabled` | warn/critical | `X-Real-IP` フォールバックを信頼すると、プロキシ設定ミスによる送信元IPスプーフィングが可能になる | `gateway.allowRealIpFallback`, `gateway.trustedProxies` | no |
| `discovery.mdns_full_mode` | warn/critical | mDNSフルモードはローカルネットワーク上で `cliPath`/`sshPort` メタデータをアドバタイズ | `discovery.mdns.mode`, `gateway.bind` | no |
| `config.insecure_or_dangerous_flags` | warn | 非セキュア/危険なデバッグフラグが有効 | 複数キー (詳細を参照) | no |
| `hooks.token_too_short` | warn | フックイングレスに対するブルートフォースが容易 | `hooks.token` | no |
| `hooks.request_session_key_enabled` | warn/critical | 外部呼び出し元がsessionKeyを選択可能 | `hooks.allowRequestSessionKey` | no |
| `hooks.request_session_key_prefixes_missing` | warn/critical | 外部セッションキーの形状に制限なし | `hooks.allowedSessionKeyPrefixes` | no |
| `logging.redact_off` | warn | 機密値がログ/ステータスに漏洩 | `logging.redactSensitive` | yes |
| `sandbox.docker_config_mode_off` | warn | サンドボックスDocker設定が存在するが非アクティブ | `agents.*.sandbox.mode` | no |
| `sandbox.dangerous_network_mode` | critical | サンドボックスDockerネットワークが `host` または `container:*` 名前空間結合モードを使用 | `agents.*.sandbox.docker.network` | no |
| `tools.exec.host_sandbox_no_sandbox_defaults` | warn | `exec host=sandbox` はサンドボックスがオフの場合ホスト実行に解決 | `tools.exec.host`, `agents.defaults.sandbox.mode` | no |
| `tools.exec.host_sandbox_no_sandbox_agents` | warn | エージェントごとの `exec host=sandbox` はサンドボックスがオフの場合ホスト実行に解決 | `agents.list[].tools.exec.host`, `agents.list[].sandbox.mode` | no |
| `tools.exec.safe_bins_interpreter_unprofiled` | warn | 明示的なプロファイルなしで `safeBins` 内のインタプリタ/ランタイムバイナリが実行リスクを広げる | `tools.exec.safeBins`, `tools.exec.safeBinProfiles`, `agents.list[].tools.exec.*` | no |
| `skills.workspace.symlink_escape` | warn | ワークスペース `skills/**/SKILL.md` がワークスペースルート外に解決 (シンボリックリンク連鎖ドリフト) | ワークスペース `skills/**` ファイルシステム状態 | no |
| `security.exposure.open_groups_with_elevated` | critical | オープングループ + 昇格ツールが高影響プロンプトインジェクションパスを作成 | `channels.*.groupPolicy`, `tools.elevated.*` | no |
| `security.exposure.open_groups_with_runtime_or_fs` | critical/warn | オープングループがサンドボックス/ワークスペースガードなしでコマンド/ファイルツールに到達可能 | `channels.*.groupPolicy`, `tools.profile/deny`, `tools.fs.workspaceOnly`, `agents.*.sandbox.mode` | no |
| `security.trust_model.multi_user_heuristic` | warn | 設定がマルチユーザーに見えるがゲートウェイ信頼モデルはパーソナルアシスタント | 信頼境界を分割、または共有ユーザー強化 (`sandbox.mode`, ツール拒否/ワークスペーススコープ) | no |
| `tools.profile_minimal_overridden` | warn | エージェントがグローバル最小プロファイルをバイパス | `agents.list[].tools.profile` | no |
| `plugins.tools_reachable_permissive_policy` | warn | 拡張ツールが寛容なコンテキストで到達可能 | `tools.profile` + ツール許可/拒否 | no |
| `models.small_params` | critical/info | 小規模モデル + 安全でないツール表面がインジェクションリスクを高める | モデル選択 + サンドボックス/ツールポリシー | no |

## HTTP経由のControl UI

Control UIは、デバイスアイデンティティを生成するために **セキュアコンテキスト** (HTTPSまたはlocalhost) を必要とします。`gateway.controlUi.allowInsecureAuth` は、セキュアコンテキスト、デバイスアイデンティティ、またはデバイスペアリングチェックを**バイパスしません**。HTTPS (Tailscale Serve) を優先するか、UIを `127.0.0.1` で開いてください。緊急時のみ、`gateway.controlUi.dangerouslyDisableDeviceAuth` がデバイスアイデンティティチェックを完全に無効にします。これは深刻なセキュリティダウングレードです。積極的にデバッグ中で迅速に戻せる場合を除き、オフに保ってください。`openclaw security audit` はこの設定が有効な場合に警告します。

## 非セキュアまたは危険なフラグの概要

`openclaw security audit` は、既知の非セキュア/危険なデバッグスイッチが有効な場合に `config.insecure_or_dangerous_flags` を含みます。現在、このチェックは以下を集約します:

-   `gateway.controlUi.allowInsecureAuth=true`
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth=true`
-   `hooks.gmail.allowUnsafeExternalContent=true`
-   `hooks.mappings[].allowUnsafeExternalContent=true`
-   `tools.exec.applyPatch.workspaceOnly=false`

OpenClaw設定スキーマで定義された完全な `dangerous*` / `dangerously*` 設定キー:

-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback`
-   `gateway.controlUi.dangerouslyDisableDeviceAuth`
-   `browser.ssrfPolicy.dangerouslyAllowPrivateNetwork`
-   `channels.discord.dangerouslyAllowNameMatching`
-   `channels.discord.accounts..dangerouslyAllowNameMatching`
-   `channels.slack.dangerouslyAllowNameMatching`
-   `channels.slack.accounts..dangerouslyAllowNameMatching`
-   `channels.googlechat.dangerouslyAllowNameMatching`
-   `channels.googlechat.accounts..dangerouslyAllowNameMatching`
-   `channels.msteams.dangerouslyAllowNameMatching`
-   `channels.irc.dangerouslyAllowNameMatching` (拡張チャネル)
-   `channels.irc.accounts..dangerouslyAllowNameMatching` (拡張チャネル)
-   `channels.mattermost.dangerouslyAllowNameMatching` (拡張チャネル)
-   `channels.mattermost.accounts..dangerouslyAllowNameMatching` (拡張チャネル)
-   `agents.defaults.sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.defaults.sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.defaults.sandbox.docker.dangerouslyAllowContainerNamespaceJoin`
-   `agents.list[].sandbox.docker.dangerouslyAllowReservedContainerTargets`
-   `agents.list[].sandbox.docker.dangerouslyAllowExternalBindSources`
-   `agents.list[].sandbox.docker.dangerouslyAllowContainerNamespaceJoin`

## リバースプロキシ設定

ゲートウェイをリバースプロキシ (nginx, Caddy, Traefikなど) の背後で実行する場合は、適切なクライアントIP検出のために `gateway.trustedProxies` を設定する必要があります。ゲートウェイが **`trustedProxies` にない** アドレスからのプロキシヘッダーを検出すると、接続をローカルクライアントとして**扱いません**。ゲートウェイ認証が無効な場合、それらの接続は拒否されます。これにより、プロキシされた接続がローカルホストからのように見えて自動的に信頼される認証バイパスを防ぎます。

```
gateway:
  trustedProxies:
    - "127.0.0.1" # プロキシがlocalhostで実行される場合
  # オプション。デフォルト false。
  # プロキシが X-Forwarded-For を提供できない場合のみ有効化。
  allowRealIpFallback: false
  auth:
    mode: password
    password: ${OPENCLAW_GATEWAY_PASSWORD}
```

`trustedProxies` が設定されている場合、ゲートウェイはクライアントIPを決定するために `X-Forwarded-For` を使用します。`X-Real-IP` は、`gateway.allowRealIpFallback: true` が明示的に設定されない限り、デフォルトでは無視されます。良好なリバースプロキシの動作 (着信転送ヘッダーを上書き):

```bash
proxy_set_header X-Forwarded-For $remote_addr;
proxy_set_header X-Real-IP $remote_addr;
```

悪いリバースプロキシの動作 (信頼できない転送ヘッダーを追加/保持):

```bash
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
```

## HSTSとオリジンノート

-   OpenClawゲートウェイはローカル/ループバック優先です。リバースプロキシでTLSを終端する場合は、プロキシに面したHTTPSドメインにHSTSを設定してください。
-   ゲートウェイ自体がHTTPSを終端する場合は、`gateway.http.securityHeaders.strictTransportSecurity` を設定して、OpenClawレスポンスからHSTSヘッダーを出力できます。
-   詳細なデプロイメントガイダンスは [信頼されたプロキシ認証](./trusted-proxy-auth.md#tls-termination-and-hsts) にあります。
-   非ループバックControl UIデプロイメントの場合、`gateway.controlUi.allowedOrigins` がデフォルトで必要です。
-   `gateway.controlUi.dangerouslyAllowHostHeaderOriginFallback=true` はHostヘッダーオリジンフォールバックモードを有効にします。危険なオペレーター選択ポリシーとして扱ってください。
-   DNSリバインデ