

  CLI コマンド

  
# security

セキュリティツール（監査 + オプションの修正）。関連項目:

-   セキュリティガイド: [セキュリティ](../gateway/security.md)

## 監査

```bash
openclaw security audit
openclaw security audit --deep
openclaw security audit --fix
openclaw security audit --json
```

この監査は、複数のDM送信者がメインセッションを共有している場合に警告を発し、共有受信トレイの強化のための**セキュアDMモード**：`session.dmScope="per-channel-peer"`（またはマルチアカウントチャネルの場合は`per-account-channel-peer`）を推奨します。これは協調的/共有受信トレイの強化のためです。相互に信頼できない/敵対的なオペレーターによって共有される単一のゲートウェイは推奨される設定ではありません。信頼境界を分離するには、別々のゲートウェイ（または別々のOSユーザー/ホスト）を使用してください。また、設定が共有ユーザーイングレスの可能性を示唆する場合（例えば、オープンなDM/グループポリシー、設定されたグループターゲット、ワイルドカード送信者ルールなど）に`security.trust_model.multi_user_heuristic`を出力し、OpenClawがデフォルトでは個人アシスタント信頼モデルであることを思い出させます。意図的な共有ユーザー設定の場合、監査ガイダンスはすべてのセッションをサンドボックス化し、ファイルシステムアクセスをワークスペーススコープに保ち、個人/プライベートなアイデンティティや認証情報をそのランタイムから外しておくことです。また、サンドボックス化なしで小さなモデル（`<=300B`）が使用され、かつWeb/ブラウザーツールが有効になっている場合にも警告します。Webhookイングレスの場合、`hooks.defaultSessionKey`が設定されていないとき、リクエスト`sessionKey`のオーバーライドが有効になっているとき、およびオーバーライドが`hooks.allowedSessionKeyPrefixes`なしで有効になっているときに警告します。また、サンドボックスモードがオフのときにサンドボックスDocker設定が構成されている場合、`gateway.nodes.denyCommands`が効果のないパターン風/不明なエントリを使用している場合（正確なノードコマンド名マッチングのみ、シェルテキストフィルタリングではありません）、`gateway.nodes.allowCommands`が危険なノードコマンドを明示的に有効にしている場合、グローバルな`tools.profile="minimal"`がエージェントツールプロファイルによって上書きされている場合、オープングループがサンドボックス/ワークスペースガードなしでランタイム/ファイルシステムツールを公開している場合、およびインストールされた拡張プラグインツールが寛容なツールポリシーの下で到達可能である場合に警告します。また、`gateway.allowRealIpFallback=true`（プロキシが誤って設定されている場合のヘッダースプーフィングリスク）と`discovery.mdns.mode="full"`（mDNS TXTレコードを介したメタデータ漏洩）にもフラグを立てます。また、サンドボックスブラウザが`sandbox.browser.cdpSourceRange`なしでDocker `bridge`ネットワークを使用している場合にも警告します。また、危険なサンドボックスDockerネットワークモード（`host`や`container:*`名前空間結合を含む）にもフラグを立てます。また、既存のサンドボックスブラウザDockerコンテナに欠落/古いハッシュラベルがある場合（例えば、移行前のコンテナで`openclaw.browserConfigEpoch`が欠落しているなど）に警告し、`openclaw sandbox recreate --browser --all`を推奨します。また、npmベースのプラグイン/フックインストールレコードが固定されていない、整合性メタデータが欠落している、または現在インストールされているパッケージバージョンからずれている場合にも警告します。チャネル許可リストが安定したIDではなく変更可能な名前/メール/タグに依存している場合（該当する場合のDiscord、Slack、Google Chat、MS Teams、Mattermost、IRCスコープ）に警告します。また、`gateway.auth.mode="none"`がゲートウェイHTTP APIを共有シークレットなしで到達可能にしている場合（`/tools/invoke`および有効な`/v1/*`エンドポイント）に警告します。`dangerous`/`dangerously`で始まる設定は、明示的なブレイクグラスオペレーターオーバーライドです。これを有効にすること自体は、セキュリティ脆弱性レポートではありません。危険なパラメータの完全なインベントリについては、[セキュリティ](../gateway/security.md)の「安全でないまたは危険なフラグの概要」セクションを参照してください。

## JSON出力

CI/ポリシーチェックには`--json`を使用します:

```bash
openclaw security audit --json | jq '.summary'
openclaw security audit --deep --json | jq '.findings[] | select(.severity=="critical") | .checkId'
```

`--fix`と`--json`を組み合わせた場合、出力には修正アクションと最終レポートの両方が含まれます:

```bash
openclaw security audit --fix --json | jq '{fix: .fix.ok, summary: .report.summary}'
```

## --fixが変更するもの

`--fix`は安全で決定的な修復を適用します:

-   一般的な`groupPolicy="open"`を`groupPolicy="allowlist"`に切り替えます（サポートされているチャネルのアカウントバリアントを含む）
-   `logging.redactSensitive`を`"off"`から`"tools"`に設定します
-   状態/設定および一般的な機密ファイル（`credentials/*.json`、`auth-profiles.json`、`sessions.json`、セッション`*.jsonl`）の権限を強化します

`--fix`は**行いません**:

-   トークン/パスワード/APIキーのローテーション
-   ツールの無効化（`gateway`、`cron`、`exec`など）
-   ゲートウェイのバインド/認証/ネットワーク公開の選択の変更
-   プラグイン/スキルの削除または書き換え

[secrets](./secrets.md)[sessions](./sessions.md)