

  セッションとメモリ

  
# セッション管理

OpenClawは、**エージェントごとの1つのダイレクトチャットセッション**をプライマリとして扱います。ダイレクトチャットは `agent::` (デフォルトは `main`) に統合され、グループ/チャンネルチャットはそれぞれ独自のキーを持ちます。`session.mainKey` が尊重されます。**ダイレクトメッセージ**のグループ化方法を制御するには `session.dmScope` を使用します:

-   `main` (デフォルト): すべてのDMが継続性のためにメインセッションを共有します。
-   `per-peer`: チャンネルをまたいで送信者IDごとに分離します。
-   `per-channel-peer`: チャンネル + 送信者ごとに分離します (マルチユーザーインボックスに推奨)。
-   `per-account-channel-peer`: アカウント + チャンネル + 送信者ごとに分離します (マルチアカウントインボックスに推奨)。`per-peer`、`per-channel-peer`、`per-account-channel-peer` を使用する際に、同じ人物がチャンネルをまたいでDMセッションを共有できるようにするには、`session.identityLinks` を使用してプロバイダー接頭辞付きのピアIDを正規のIDにマッピングします。

## セキュアDMモード (マルチユーザー設定に推奨)

> **セキュリティ警告:** あなたのエージェントが**複数の人**からDMを受信できる場合、セキュアDMモードを有効にすることを強く検討すべきです。これがないと、すべてのユーザーが同じ会話コンテキストを共有し、ユーザー間でプライベートな情報が漏洩する可能性があります。

**デフォルト設定での問題例:**

-   アリス (`<SENDER_A>`) があなたのエージェントにプライベートな話題 (例えば、医療予約) についてメッセージを送ります。
-   ボブ (`<SENDER_B>`) があなたのエージェントに「私たちは何について話していましたか？」と尋ねます。
-   両方のDMが同じセッションを共有しているため、モデルはアリスの以前のコンテキストを使ってボブに回答する可能性があります。

**解決策:** `dmScope` を設定して、ユーザーごとにセッションを分離します:

```
// ~/.openclaw/openclaw.json
{
  session: {
    // セキュアDMモード: DMコンテキストをチャンネル + 送信者ごとに分離。
    dmScope: "per-channel-peer",
  },
}
```

**これを有効にするべき状況:**

-   複数の送信者に対してペアリング承認がある場合
-   複数のエントリを持つDM許可リストを使用している場合
-   `dmPolicy: "open"` を設定している場合
-   複数の電話番号やアカウントがあなたのエージェントにメッセージを送れる場合

注意点:

-   デフォルトは継続性のため `dmScope: "main"` です (すべてのDMがメインセッションを共有)。これはシングルユーザー設定では問題ありません。
-   ローカルCLIオンボーディングでは、未設定時にデフォルトで `session.dmScope: "per-channel-peer"` を書き込みます (既存の明示的な値は保持されます)。
-   同じチャンネル上のマルチアカウントインボックスの場合は、`per-account-channel-peer` を優先します。
-   同じ人物が複数のチャンネルであなたに連絡する場合は、`session.identityLinks` を使用して彼らのDMセッションを1つの正規IDに統合します。
-   DM設定は `openclaw security audit` で確認できます ([セキュリティ](../cli/security.md) を参照)。

## ゲートウェイが信頼できる情報源

すべてのセッション状態は**ゲートウェイ** (「マスター」OpenClaw) によって**所有されています**。UIクライアント (macOSアプリ、WebChatなど) は、ローカルファイルを読み取る代わりに、セッションリストやトークンカウントをゲートウェイに問い合わせる必要があります。

-   **リモートモード**では、重要なセッションストアはリモートゲートウェイホスト上にあり、あなたのMac上にはありません。
-   UIに表示されるトークンカウントは、ゲートウェイのストアフィールド (`inputTokens`、`outputTokens`、`totalTokens`、`contextTokens`) から取得されます。クライアントはJSONLトランスクリプトを解析して合計を「修正」することはありません。

## 状態の保存場所

-   **ゲートウェイホスト**上:
    -   ストアファイル: `~/.openclaw/agents//sessions/sessions.json` (エージェントごと)。
-   トランスクリプト: `~/.openclaw/agents//sessions/.jsonl` (Telegramトピックセッションは `.../-topic-.jsonl` を使用)。
-   ストアは `sessionKey -> { sessionId, updatedAt, ... }` のマップです。エントリを削除しても安全です。必要に応じて再作成されます。
-   グループエントリには、UIでセッションにラベルを付けるための `displayName`、`channel`、`subject`、`room`、`space` が含まれる場合があります。
-   セッションエントリには、UIがセッションの出所を説明できるようにするための `origin` メタデータ (ラベル + ルーティングヒント) が含まれます。
-   OpenClawは**レガシーなPi/Tauセッションフォルダを読み取りません**。

## メンテナンス

OpenClawはセッションストアメンテナンスを適用し、`sessions.json` とトランスクリプトアーティファクトが時間の経過とともに制限されるようにします。

### デフォルト

-   `session.maintenance.mode`: `warn`
-   `session.maintenance.pruneAfter`: `30d`
-   `session.maintenance.maxEntries`: `500`
-   `session.maintenance.rotateBytes`: `10mb`
-   `session.maintenance.resetArchiveRetention`: デフォルトは `pruneAfter` (`30d`)
-   `session.maintenance.maxDiskBytes`: 未設定 (無効)
-   `session.maintenance.highWaterBytes`: 予算管理が有効な場合、デフォルトは `maxDiskBytes` の `80%`

### 仕組み

メンテナンスはセッションストア書き込み中に実行され、`openclaw sessions cleanup` でオンデマンドでトリガーすることもできます。

-   `mode: "warn"`: 削除される内容を報告しますが、エントリ/トランスクリプトを変更しません。
-   `mode: "enforce"`: 以下の順序でクリーンアップを適用します:
    1.  `pruneAfter` より古い古いエントリを剪定
    2.  エントリ数を `maxEntries` に制限 (最も古いものから)
    3.  参照されなくなった削除済みエントリのトランスクリプトファイルをアーカイブ
    4.  保持ポリシーに従って古い `*.deleted.` および `*.reset.` アーカイブを削除
    5.  `sessions.json` が `rotateBytes` を超えた場合にローテート
    6.  `maxDiskBytes` が設定されている場合、ディスク予算を `highWaterBytes` に向けて強制 (最も古いアーティファクトから、次に最も古いセッション)

### 大規模ストアのパフォーマンスに関する注意点

大規模なセッションストアは、高ボリューム設定では一般的です。メンテナンス作業は書き込みパスの作業であるため、非常に大規模なストアは書き込み遅延を増加させる可能性があります。コストを最も増加させる要因:

-   非常に高い `session.maintenance.maxEntries` 値
-   古いエントリを保持する長い `pruneAfter` ウィンドウ
-   `~/.openclaw/agents//sessions/` 内の多数のトランスクリプト/アーカイブアーティファクト
-   合理的な剪定/上限制限なしにディスク予算 (`maxDiskBytes`) を有効にする

対処法:

-   本番環境では `mode: "enforce"` を使用して、成長が自動的に制限されるようにする
-   時間制限とカウント制限の両方 (`pruneAfter` + `maxEntries`) を設定する (片方だけではない)
-   大規模なデプロイメントでは、厳格な上限のために `maxDiskBytes` + `highWaterBytes` を設定する
-   `highWaterBytes` を `maxDiskBytes` よりも有意に低く保つ (デフォルトは80%)
-   設定変更後、強制実行前に予測される影響を確認するために `openclaw sessions cleanup --dry-run --json` を実行する
-   頻繁にアクティブなセッションの場合は、手動クリーンアップ実行時に `--active-key` を渡す

### カスタマイズ例

保守的な強制ポリシーを使用:

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "45d",
      maxEntries: 800,
      rotateBytes: "20mb",
      resetArchiveRetention: "14d",
    },
  },
}
```

セッションディレクトリの厳格なディスク予算を有効化:

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      maxDiskBytes: "1gb",
      highWaterBytes: "800mb",
    },
  },
}
```

大規模インストール向けに調整 (例):

```json
{
  session: {
    maintenance: {
      mode: "enforce",
      pruneAfter: "14d",
      maxEntries: 2000,
      rotateBytes: "25mb",
      maxDiskBytes: "2gb",
      highWaterBytes: "1.6gb",
    },
  },
}
```

CLIからメンテナンスをプレビューまたは強制実行:

```bash
openclaw sessions cleanup --dry-run
openclaw sessions cleanup --enforce
```

## セッション剪定

OpenClawはデフォルトで、LLM呼び出しの直前に**メモリ内コンテキストから古いツール結果**を削除します。これはJSONL履歴を**書き換えません**。[/concepts/session-pruning](./session-pruning.md) を参照。

## 事前圧縮メモリフラッシュ

セッションが自動圧縮に近づくと、OpenClawはモデルに永続的なメモをディスクに書き込むよう促す**サイレントメモリフラッシュ**ターンを実行できます。これはワークスペースが書き込み可能な場合にのみ実行されます。[メモリ](./memory.md) と [圧縮](./compaction.md) を参照。

## トランスポート → セッションキーのマッピング

-   ダイレクトチャットは `session.dmScope` に従います (デフォルト `main`)。
    -   `main`: `agent::` (デバイス/チャンネル間での継続性)。
        -   複数の電話番号やチャンネルが同じエージェントメインキーにマッピングできます。それらは1つの会話へのトランスポートとして機能します。
    -   `per-peer`: `agent::dm:`。
    -   `per-channel-peer`: `agent:::dm:`。
    -   `per-account-channel-peer`: `agent::::dm:` (accountId のデフォルトは `default`)。
    -   `session.identityLinks` がプロバイダー接頭辞付きのピアID (例: `telegram:123`) に一致する場合、正規キーが `` を置き換え、同じ人物がチャンネルをまたいでセッションを共有します。
-   グループチャットは状態を分離: `agent:::group:` (ルーム/チャンネルは `agent:::channel:` を使用)。
    -   Telegramフォーラムトピックは、分離のためにグループIDに `:topic:` を追加します。
    -   レガシーな `group:` キーは移行のためにまだ認識されます。
-   インバウンドコンテキストはまだ `group:` を使用する場合があります。チャンネルは `Provider` から推測され、正規の `agent:::group:` 形式に正規化されます。
-   その他のソース:
    -   クロンジョブ: `cron:<job.id>`
    -   Webhooks: `hook:` (フックで明示的に設定されていない限り)
    -   ノード実行: `node-`

## ライフサイクル

-   リセットポリシー: セッションは期限切れになるまで再利用され、期限切れは次のインバウンドメッセージで評価されます。
-   日次リセット: デフォルトは**ゲートウェイホストの現地時間で午前4:00**。セッションの最終更新が最新の日次リセット時間より前の場合、そのセッションは古くなります。
-   アイドルリセット (オプション): `idleMinutes` はスライディングアイドルウィンドウを追加します。日次リセットとアイドルリセットの両方が設定されている場合、**どちらかが先に期限切れになると**新しいセッションが強制されます。
-   レガシーなアイドルのみ: `session.reset`/`resetByType` 設定なしで `session.idleMinutes` を設定した場合、OpenClawは後方互換性のためにアイドルのみモードのままです。
-   タイプごとのオーバーライド (オプション): `resetByType` を使用して、`direct`、`group`、`thread` セッション (スレッド = Slack/Discordスレッド、Telegramトピック、コネクタによって提供される場合のMatrixスレッド) のポリシーをオーバーライドできます。
-   チャンネルごとのオーバーライド (オプション): `resetByChannel` はチャンネルのリセットポリシーをオーバーライドします (そのチャンネルのすべてのセッションタイプに適用され、`reset`/`resetByType` より優先されます)。
-   リセットトリガー: 正確な `/new` または `/reset` (および `resetTriggers` 内の追加トリガー) は新しいセッションIDを開始し、メッセージの残りを通過させます。`/new ` はモデルエイリアス、`provider/model`、またはプロバイダー名 (あいまい一致) を受け入れ、新しいセッションモデルを設定します。`/new` または `/reset` が単独で送信された場合、OpenClawはリセットを確認する短い「hello」挨拶ターンを実行します。
-   手動リセット: ストアから特定のキーを削除するか、JSONLトランスクリプトを削除します。次のメッセージでそれらが再作成されます。
-   分離されたクロンジョブは常に実行ごとに新しい `sessionId` を作成します (アイドル再利用なし)。

## 送信ポリシー (オプション)

個々のIDをリストアップせずに、特定のセッションタイプの配信をブロックします。

```json
{
  session: {
    sendPolicy: {
      rules: [
        { action: "deny", match: { channel: "discord", chatType: "group" } },
        { action: "deny", match: { keyPrefix: "cron:" } },
        // 生のセッションキー (`agent:<id>:` 接頭辞を含む) に一致。
        { action: "deny", match: { rawKeyPrefix: "agent:main:discord:" } },
      ],
      default: "allow",
    },
  },
}
```

ランタイムオーバーライド (所有者のみ):

-   `/send on` → このセッションに対して許可
-   `/send off` → このセッションに対して拒否
-   `/send inherit` → オーバーライドをクリアし、設定ルールを使用
これらは登録されるように、スタンドアロンメッセージとして送信してください。

## 設定 (オプションのリネーム例)

```
// ~/.openclaw/openclaw.json
{
  session: {
    scope: "per-sender", // グループキーを分離して保持
    dmScope: "main", // DM継続性 (共有インボックスの場合は per-channel-peer/per-account-channel-peer を設定)
    identityLinks: {
      alice: ["telegram:123456789", "discord:987654321012345678"],
    },
    reset: {
      // デフォルト: mode=daily, atHour=4 (ゲートウェイホストの現地時間)。
      // idleMinutes も設定した場合、どちらかが先に期限切れになった方が優先されます。
      mode: "daily",
      atHour: 4,
      idleMinutes: 120,
    },
    resetByType: {
      thread: { mode: "daily", atHour: 4 },
      direct: { mode: "idle", idleMinutes: 240 },
      group: { mode: "idle", idleMinutes: 120 },
    },
    resetByChannel: {
      discord: { mode: "idle", idleMinutes: 10080 },
    },
    resetTriggers: ["/new", "/reset"],
    store: "~/.openclaw/agents/{agentId}/sessions/sessions.json",
    mainKey: "main",
  },
}
```

## 検査

-   `openclaw status` — ストアパスと最近のセッションを表示。
-   `openclaw sessions --json` — すべてのエントリをダンプ (`--active ` でフィルタ)。
-   `openclaw gateway call sessions.list --params '{}'` — 実行中のゲートウェイからセッションを取得 (リモートゲートウェイアクセスの場合は `--url`/`--token` を使用)。
-   チャットで `/status` をスタンドアロンメッセージとして送信して、エージェントが到達可能かどうか、セッションコンテキストがどの程度使用されているか、現在の思考/詳細表示の切り替え状態、WhatsApp Web認証情報が最後に更新された時期 (再リンクの必要性を確認するのに役立つ) を確認します。
-   `/context list` または `/context detail` を送信して、システムプロンプトと注入されたワークスペースファイル (および最大のコンテキスト貢献者) に何が含まれているかを確認します。
-   `/stop` (または `stop`、`stop action`、`stop run`、`stop openclaw` などのスタンドアロン中止フレーズ) を送信して、現在の実行を中止し、そのセッションのキューに入ったフォローアップをクリアし、そこから生成されたサブエージェントの実行を停止します (返信には停止された数が含まれます)。
-   `/compact` (オプションの指示) をスタンドアロンメッセージとして送信して、古いコンテキストを要約し、ウィンドウスペースを解放します。[/concepts/compaction](./compaction.md) を参照。
-   JSONLトランスクリプトは直接開いて完全なターンを確認できます。

## ヒント

-   プライマリキーは1:1トラフィック専用に保ち、グループには独自のキーを持たせます。
-   クリーンアップを自動化する際は、他の場所のコンテキストを保持するために、ストア全体ではなく個々のキーを削除します。

## セッション出所メタデータ

各セッションエントリは、その出所を (ベストエフォートで) `origin` に記録します:

-   `label`: 人間が読めるラベル (会話ラベル + グループ件名/チャンネルから解決)
-   `provider`: 正規化されたチャンネルID (拡張機能を含む)
-   `from`/`to`: インバウンドエンベロープからの生のルーティングID
-   `accountId`: プロバイダーアカウントID (マルチアカウントの場合)
-   `threadId`: チャンネルがサポートする場合のスレッド/トピックID
originフィールドは、ダイレクトメッセージ、チャンネル、グループに対して設定されます。コネクタが配信ルーティングのみを更新する場合 (例えば、DMメインセッションを新鮮に保つため)、セッションがその説明メタデータを保持できるように、インバウンドコンテキストを提供する必要があります。拡張機能は、インバウンドコンテキストで `ConversationLabel`、`GroupSubject`、`GroupChannel`、`GroupSpace`、`SenderName` を送信し、`recordSessionMetaFromInbound` を呼び出す (または同じコンテキストを `updateLastRoute` に渡す) ことでこれを行えます。

[ブートストラップ](../start/bootstrapping.md)[セッション剪定](./session-pruning.md)