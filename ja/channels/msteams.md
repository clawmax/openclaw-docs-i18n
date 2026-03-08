

  メッセージングプラットフォーム

  
# Microsoft Teams

> 「ここに入る者は一切の希望を捨てよ」

更新日: 2026-01-21 ステータス: テキスト + DM 添付ファイルはサポート済み。チャネル/グループでのファイル送信には `sharePointSiteId` + Graph 権限が必要（[グループチャットでのファイル送信](#sending-files-in-group-chats) を参照）。投票は Adaptive Cards 経由で送信されます。

## 必要なプラグイン

Microsoft Teams はプラグインとして提供され、コアインストールにはバンドルされていません。**破壊的変更 (2026.1.15):** MS Teams はコアから移動しました。使用する場合は、プラグインをインストールする必要があります。説明: コアインストールを軽量に保ち、MS Teams の依存関係を独立して更新できるようにするためです。CLI (npm レジストリ) 経由でインストール:

```bash
openclaw plugins install @openclaw/msteams
```

ローカルチェックアウト (git リポジトリから実行する場合):

```bash
openclaw plugins install ./extensions/msteams
```

configure/オンボーディング中に Teams を選択し、git チェックアウトが検出された場合、OpenClaw は自動的にローカルインストールパスを提供します。詳細: [プラグイン](../tools/plugin.md)

## クイックセットアップ (初心者向け)

1.  Microsoft Teams プラグインをインストールします。
2.  **Azure Bot** (アプリ ID + クライアントシークレット + テナント ID) を作成します。
3.  これらの認証情報で OpenClaw を設定します。
4.  `/api/messages` (デフォルトはポート 3978) を公開 URL またはトンネル経由で公開します。
5.  Teams アプリパッケージをインストールし、ゲートウェイを起動します。

最小構成:

```json
{
  channels: {
    msteams: {
      enabled: true,
      appId: "<APP_ID>",
      appPassword: "<APP_PASSWORD>",
      tenantId: "<TENANT_ID>",
      webhook: { port: 3978, path: "/api/messages" },
    },
  },
}
```

注: グループチャットはデフォルトでブロックされます (`channels.msteams.groupPolicy: "allowlist"`)。グループ返信を許可するには、`channels.msteams.groupAllowFrom` を設定します (または `groupPolicy: "open"` を使用して、メンションゲートされた任意のメンバーを許可します)。

## 目標

-   Teams DM、グループチャット、またはチャネル経由で OpenClaw と会話します。
-   ルーティングを確定的に保つ: 返信は常に到着したチャネルに戻ります。
-   安全なチャネル動作をデフォルトとする (設定しない限りメンションが必要)。

## 設定書き込み

デフォルトでは、Microsoft Teams は `/config set|unset` によってトリガーされる設定更新の書き込みを許可します (`commands.config: true` が必要)。無効にするには:

```json
{
  channels: { msteams: { configWrites: false } },
}
```

## アクセス制御 (DM + グループ)

**DM アクセス**

-   デフォルト: `channels.msteams.dmPolicy = "pairing"`。未知の送信者は承認されるまで無視されます。
-   `channels.msteams.allowFrom` は安定した AAD オブジェクト ID を使用する必要があります。
-   UPN/表示名は変更可能です。直接マッチングはデフォルトで無効化されており、`channels.msteams.dangerouslyAllowNameMatching: true` でのみ有効になります。
-   ウィザードは、認証情報が許可する場合、Microsoft Graph 経由で名前を ID に解決できます。

**グループアクセス**

-   デフォルト: `channels.msteams.groupPolicy = "allowlist"` (`groupAllowFrom` を追加しない限りブロックされます)。未設定時のデフォルトを上書きするには `channels.defaults.groupPolicy` を使用します。
-   `channels.msteams.groupAllowFrom` は、グループチャット/チャネルでトリガーできる送信者を制御します (`channels.msteams.allowFrom` にフォールバック)。
-   `groupPolicy: "open"` を設定すると、任意のメンバーを許可します (デフォルトでは依然としてメンションゲートされます)。
-   **チャネルを一切許可しない** 場合は、`channels.msteams.groupPolicy: "disabled"` を設定します。

例:

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      groupAllowFrom: ["user@org.com"],
    },
  },
}
```

**Teams + チャネル許可リスト**

-   `channels.msteams.teams` の下にチームとチャネルをリストすることで、グループ/チャネル返信のスコープを制限します。
-   キーはチーム ID または名前です。チャネルキーは会話 ID または名前です。
-   `groupPolicy="allowlist"` でチーム許可リストが存在する場合、リストされたチーム/チャネルのみが受け入れられます (メンションゲート)。
-   設定ウィザードは `Team/Channel` エントリを受け入れ、保存します。
-   起動時、OpenClaw はチーム/チャネルおよびユーザー許可リストの名前を ID に解決します (Graph 権限が許可する場合) し、マッピングをログに記録します。解決できないエントリは入力されたまま保持されます。

例:

```json
{
  channels: {
    msteams: {
      groupPolicy: "allowlist",
      teams: {
        "My Team": {
          channels: {
            General: { requireMention: true },
          },
        },
      },
    },
  },
}
```

## 仕組み

1.  Microsoft Teams プラグインをインストールします。
2.  **Azure Bot** (アプリ ID + シークレット + テナント ID) を作成します。
3.  ボットを参照し、以下の RSC 権限を含む **Teams アプリパッケージ** を構築します。
4.  Teams アプリをチーム (または DM 用の個人スコープ) にアップロード/インストールします。
5.  `~/.openclaw/openclaw.json` (または環境変数) で `msteams` を設定し、ゲートウェイを起動します。
6.  ゲートウェイは、デフォルトで `/api/messages` で Bot Framework の Webhook トラフィックをリッスンします。

## Azure Bot セットアップ (前提条件)

OpenClaw を設定する前に、Azure Bot リソースを作成する必要があります。

### ステップ 1: Azure Bot の作成

1.  [Azure Bot の作成](https://portal.azure.com/#create/Microsoft.AzureBot) に移動します。
2.  **基本** タブを入力します:
    
    | フィールド | 値 |
    | --- | --- |
    | **ボットハンドル** | ボット名、例: `openclaw-msteams` (一意である必要があります) |
    | **サブスクリプション** | Azure サブスクリプションを選択 |
    | **リソースグループ** | 新規作成または既存のものを使用 |
    | **価格レベル** | 開発/テスト用は **Free** |
    | **アプリの種類** | **シングルテナント** (推奨 - 以下の注記を参照) |
    | **作成タイプ** | **新しい Microsoft アプリ ID を作成** |
    

> **非推奨通知:** 新しいマルチテナントボットの作成は 2025-07-31 以降非推奨となりました。新しいボットには **シングルテナント** を使用してください。

3.  **レビュー + 作成** → **作成** をクリックします (約 1-2 分待機)

### ステップ 2: 認証情報の取得

1.  Azure Bot リソース → **構成** に移動します。
2.  **Microsoft アプリ ID** をコピー → これが `appId` です。
3.  **パスワードの管理** をクリック → アプリ登録に移動します。
4.  **証明書とシークレット** → **新しいクライアントシークレット** → **値** をコピー → これが `appPassword` です。
5.  **概要** → **ディレクトリ (テナント) ID** をコピー → これが `tenantId` です。

### ステップ 3: メッセージングエンドポイントの設定

1.  Azure Bot → **構成** で、
2.  **メッセージングエンドポイント** を Webhook URL に設定します:
    -   本番環境: `https://your-domain.com/api/messages`
    -   ローカル開発: トンネルを使用します (以下の [ローカル開発 (トンネリング)](#local-development-tunneling) を参照)

### ステップ 4: Teams チャネルの有効化

1.  Azure Bot → **チャネル** で、
2.  **Microsoft Teams** → 構成 → 保存 をクリックします。
3.  利用規約に同意します。

## ローカル開発 (トンネリング)

Teams は `localhost` に到達できません。ローカル開発にはトンネルを使用します: **オプション A: ngrok**

```bash
ngrok http 3978
# https URL をコピー、例: https://abc123.ngrok.io
# メッセージングエンドポイントを設定: https://abc123.ngrok.io/api/messages
```

**オプション B: Tailscale Funnel**

```bash
tailscale funnel 3978
# Tailscale funnel URL をメッセージングエンドポイントとして使用
```

## Teams デベロッパーポータル (代替方法)

マニフェスト ZIP を手動で作成する代わりに、[Teams デベロッパーポータル](https://dev.teams.microsoft.com/apps) を使用できます:

1.  **\+ 新しいアプリ** をクリックします。
2.  基本情報 (名前、説明、開発者情報) を入力します。
3.  **アプリ機能** → **ボット** に移動します。
4.  **ボット ID を手動で入力** を選択し、Azure Bot アプリ ID を貼り付けます。
5.  スコープを確認: **個人用**、**チーム**、**グループチャット**。
6.  **配布** → **アプリパッケージのダウンロード** をクリックします。
7.  Teams で: **アプリ** → **アプリの管理** → **カスタムアプリをアップロード** → ZIP を選択します。

これは JSON マニフェストを手動で編集するよりも簡単なことが多いです。

## ボットのテスト

**オプション A: Azure Web Chat (まず Webhook を確認)**

1.  Azure Portal → Azure Bot リソース → **Web チャットでテスト** で、
2.  メッセージを送信 - 応答が表示されるはずです。
3.  これにより、Teams セットアップ前に Webhook エンドポイントが機能していることが確認されます。

**オプション B: Teams (アプリインストール後)**

1.  Teams アプリをインストールします (サイドロードまたは組織カタログ)。
2.  Teams でボットを見つけ、DM を送信します。
3.  受信アクティビティのゲートウェイログを確認します。

## セットアップ (最小限のテキストのみ)

1.  **Microsoft Teams プラグインのインストール**
    -   npm から: `openclaw plugins install @openclaw/msteams`
    -   ローカルチェックアウトから: `openclaw plugins install ./extensions/msteams`
2.  **ボット登録**
    -   Azure Bot を作成し (上記参照)、以下をメモします:
        -   アプリ ID
        -   クライアントシークレット (アプリパスワード)
        -   テナント ID (シングルテナント)
3.  **Teams アプリマニフェスト**
    -   `botId = ` の `bot` エントリを含めます。
    -   スコープ: `personal`、`team`、`groupChat`。
    -   `supportsFiles: true` (個人スコープのファイル処理に必要)。
    -   RSC 権限を追加します (下記)。
    -   アイコンを作成: `outline.png` (32x32) と `color.png` (192x192)。
    -   3つのファイルをまとめて ZIP 化: `manifest.json`、`outline.png`、`color.png`。
4.  **OpenClaw の設定**
    
    コピー
    
    ```json
    {
      "msteams": {
        "enabled": true,
        "appId": "<APP_ID>",
        "appPassword": "<APP_PASSWORD>",
        "tenantId": "<TENANT_ID>",
        "webhook": { "port": 3978, "path": "/api/messages" }
      }
    }
    ```
    
    設定キーの代わりに環境変数も使用できます:
    -   `MSTEAMS_APP_ID`
    -   `MSTEAMS_APP_PASSWORD`
    -   `MSTEAMS_TENANT_ID`
5.  **ボットエンドポイント**
    -   Azure Bot メッセージングエンドポイントを以下に設定します:
        -   `https://:3978/api/messages` (または選択したパス/ポート)。
6.  **ゲートウェイの実行**
    -   Teams チャネルは、プラグインがインストールされ、`msteams` 設定に認証情報が存在する場合、自動的に起動します。

## 履歴コンテキスト

-   `channels.msteams.historyLimit` は、プロンプトにラップされる最近のチャネル/グループメッセージの数を制御します。
-   `messages.groupChat.historyLimit` にフォールバックします。無効にするには `0` を設定します (デフォルト 50)。
-   DM 履歴は `channels.msteams.dmHistoryLimit` (ユーザーターン) で制限できます。ユーザーごとの上書き: `channels.msteams.dms["<user_id>"].historyLimit`。

## 現在の Teams RSC 権限 (マニフェスト)

これらは、Teams アプリマニフェスト内の **既存のリソース固有の権限** です。これらはアプリがインストールされているチーム/チャット内でのみ適用されます。**チャネル (チームスコープ) の場合:**

-   `ChannelMessage.Read.Group` (アプリケーション) - @メンションなしで全チャネルメッセージを受信
-   `ChannelMessage.Send.Group` (アプリケーション)
-   `Member.Read.Group` (アプリケーション)
-   `Owner.Read.Group` (アプリケーション)
-   `ChannelSettings.Read.Group` (アプリケーション)
-   `TeamMember.Read.Group` (アプリケーション)
-   `TeamSettings.Read.Group` (アプリケーション)

**グループチャットの場合:**

-   `ChatMessage.Read.Chat` (アプリケーション) - @メンションなしで全グループチャットメッセージを受信

## Teams マニフェストの例 (編集済み)

必要なフィールドを含む最小限の有効な例。ID と URL を置き換えてください。

```json
{
  "$schema": "https://developer.microsoft.com/en-us/json-schemas/teams/v1.23/MicrosoftTeams.schema.json",
  "manifestVersion": "1.23",
  "version": "1.0.0",
  "id": "00000000-0000-0000-0000-000000000000",
  "name": { "short": "OpenClaw" },
  "developer": {
    "name": "Your Org",
    "websiteUrl": "https://example.com",
    "privacyUrl": "https://example.com/privacy",
    "termsOfUseUrl": "https://example.com/terms"
  },
  "description": { "short": "OpenClaw in Teams", "full": "OpenClaw in Teams" },
  "icons": { "outline": "outline.png", "color": "color.png" },
  "accentColor": "#5B6DEF",
  "bots": [
    {
      "botId": "11111111-1111-1111-1111-111111111111",
      "scopes": ["personal", "team", "groupChat"],
      "isNotificationOnly": false,
      "supportsCalling": false,
      "supportsVideo": false,
      "supportsFiles": true
    }
  ],
  "webApplicationInfo": {
    "id": "11111111-1111-1111-1111-111111111111"
  },
  "authorization": {
    "permissions": {
      "resourceSpecific": [
        { "name": "ChannelMessage.Read.Group", "type": "Application" },
        { "name": "ChannelMessage.Send.Group", "type": "Application" },
        { "name": "Member.Read.Group", "type": "Application" },
        { "name": "Owner.Read.Group", "type": "Application" },
        { "name": "ChannelSettings.Read.Group", "type": "Application" },
        { "name": "TeamMember.Read.Group", "type": "Application" },
        { "name": "TeamSettings.Read.Group", "type": "Application" },
        { "name": "ChatMessage.Read.Chat", "type": "Application" }
      ]
    }
  }
}
```

### マニフェストの注意点 (必須フィールド)

-   `bots[].botId` は Azure Bot アプリ ID と **一致する必要があります**。
-   `webApplicationInfo.id` は Azure Bot アプリ ID と **一致する必要があります**。
-   `bots[].scopes` は使用予定のサーフェス (`personal`、`team`、`groupChat`) を含める必要があります。
-   `bots[].supportsFiles: true` は個人スコープでのファイル処理に必要です。
-   `authorization.permissions.resourceSpecific` は、チャネルトラフィックを希望する場合、チャネルの読み取り/送信を含める必要があります。

### 既存アプリの更新

既にインストールされている Teams アプリを更新する場合 (例: RSC 権限を追加):

1.  新しい設定で `manifest.json` を更新します。
2.  **`version` フィールドを増分します** (例: `1.0.0` → `1.1.0`)。
3.  アイコン (`manifest.json`、`outline.png`、`color.png`) を含めてマニフェストを **再 ZIP 化** します。
4.  新しい ZIP をアップロードします:
    -   **オプション A (Teams 管理センター):** Teams 管理センター → Teams アプリ → アプリの管理 → アプリを検索 → 新しいバージョンをアップロード
    -   **オプション B (サイドロード):** Teams で → アプリ → アプリの管理 → カスタムアプリをアップロード
5.  **チームチャネルの場合:** 新しい権限を有効にするには、各チームでアプリを再インストールします。
6.  キャッシュされたアプリメタデータをクリアするために、Teams を **完全に終了して再起動** します (ウィンドウを閉じるだけでは不十分)。

## 機能: RSC のみ vs Graph

### Teams RSC のみの場合 (アプリインストール済み、Graph API 権限なし)

動作するもの:

-   チャネルメッセージの **テキスト** コンテンツの読み取り。
-   チャネルメッセージの **テキスト** コンテンツの送信。
-   **個人 (DM)** ファイル添付の受信。

動作しないもの:

-   チャネル/グループの **画像またはファイルコンテンツ** (ペイロードには HTML スタブのみが含まれます)。
-   SharePoint/OneDrive に保存された添付ファイルのダウンロード。
-   メッセージ履歴の読み取り (ライブ Webhook イベントを超えて)。

### Teams RSC + Microsoft Graph アプリケーション権限の場合

追加されるもの:

-   ホストされたコンテンツ (メッセージに貼り付けられた画像) のダウンロード。
-   SharePoint/OneDrive に保存されたファイル添付のダウンロード。
-   Graph 経由でのチャネル/チャットメッセージ履歴の読み取り。

### RSC vs Graph API

| 機能 | RSC 権限 | Graph API |
| --- | --- | --- |
| **リアルタイムメッセージ** | はい (Webhook 経由) | いいえ (ポーリングのみ) |
| **履歴メッセージ** | いいえ | はい (履歴をクエリ可能) |
| **セットアップの複雑さ** | アプリマニフェストのみ | 管理者の同意 + トークンフローが必要 |
| **オフライン動作** | いいえ (実行中である必要あり) | はい (いつでもクエリ可能) |

**結論:** RSC はリアルタイムリスニング用、Graph API は履歴アクセス用です。オフライン中に見逃したメッセージをキャッチアップするには、`ChannelMessage.Read.All` を持つ Graph API が必要です (管理者の同意が必要)。

## Graph 対応メディア + 履歴 (チャネルに必要)

**チャネル** での画像/ファイルが必要な場合、または **メッセージ履歴** を取得したい場合は、Microsoft Graph 権限を有効化し、管理者の同意を付与する必要があります。

1.  Entra ID (Azure AD) **アプリ登録** で、Microsoft Graph **アプリケーション権限** を追加します:
    -   `ChannelMessage.Read.All` (チャネル添付ファイル + 履歴)
    -   `Chat.Read.All` または `ChatMessage.Read.All` (グループチャット)
2.  テナントに対して **管理者の同意を付与** します。
3.  Teams アプリの **マニフェストバージョン** を上げ、再アップロードし、Teams で **アプリを再インストール** します。
4.  キャッシュされたアプリメタデータをクリアするために、Teams を **完全に終了して再起動** します。

**ユーザーメンションの追加権限:** 会話内のユーザーに対するユーザー @メンションはそのまま動作します。ただし、**現在の会話にいない** ユーザーを動的に検索してメンションしたい場合は、`User.Read.All` (アプリケーション) 権限を追加し、管理者の同意を付与します。

## 既知の制限事項

### Webhook タイムアウト

Teams は HTTP Webhook 経由でメッセージを配信します。処理に時間がかかりすぎる場合 (例: 遅い LLM 応答)、以下が発生する可能性があります:

-   ゲートウェイタイムアウト
-   Teams がメッセージを再試行 (重複の原因)
-   返信のドロップ

OpenClaw は迅速に応答を返し、返信をプロアクティブに送信することでこれを処理しますが、非常に遅い応答は依然として問題を引き起こす可能性があります。

### フォーマット

Teams のマークダウンは Slack や Discord よりも制限されています:

-   基本的なフォーマットは動作: **太字**、*斜体*、`コード`、リンク
-   複雑なマークダウン (テーブル、ネストされたリスト) は正しくレンダリングされない可能性があります
-   投票および任意のカード送信には Adaptive Cards がサポートされています (下記参照)

## 設定

主要な設定 (共有チャネルパターンについては `/gateway/configuration` を参照):

-   `channels.msteams.enabled`: チャネルの有効化/無効化。
-   `channels.msteams.appId`、`channels.msteams.appPassword`、`channels.msteams.tenantId`: ボット認証情報。
-   `channels.msteams.webhook.port` (デフォルト `3978`)
-   `channels.msteams.webhook.path` (デフォルト `/api/messages`)
-   `channels.msteams.dmPolicy`: `pairing | allowlist | open | disabled` (デフォルト: pairing)
-   `channels.msteams.allowFrom`: DM 許可リスト (AAD オブジェクト ID 推奨)。ウィザードは、セットアップ中に Graph アクセスが利用可能な場合、名前を ID に解決します。
-   `channels.msteams.dangerouslyAllowNameMatching`: 変更可能な UPN/表示名マッチングを再度有効にするブレークグラストグル。
-   `channels.msteams.textChunkLimit`: 送信テキストのチャンクサイズ。
-   `channels.msteams.chunkMode`: `length` (デフォルト) または `newline` で、長さチャンク化の前に空白行 (段落境界) で分割します。
-   `channels.msteams.mediaAllowHosts`: 受信添付ファイルホストの許可リスト (デフォルトは Microsoft/Teams ドメイン)。
-   `channels.msteams.mediaAuthAllowHosts`: メディア再試行時に Authorization ヘッダーを添付するホストの許可リスト (デフォルトは Graph + Bot Framework ホスト)。
-   `channels.msteams.requireMention`: チャネル/グループで @メンションを要求 (デフォルト true)。
-   `channels.msteams.replyStyle`: `thread | top-level` ([返信スタイル](#reply-style-threads-vs-posts) を参照)。
-   `channels.msteams.teams..replyStyle`: チームごとの上書き。
-   `channels.msteams.teams..requireMention`: チームごとの上書き。
-   `channels.msteams.teams..tools`: チャネル上書きがない場合に使用される、チームごとのデフォルトツールポリシー上書き (`allow`/`deny`/`alsoAllow`)。
-   `channels.msteams.teams..toolsBySender`: チームごとの送信者ごとのデフォルトツールポリシー上書き (`"*"` ワイルドカードサポート)。
-   `channels.msteams.teams..channels..replyStyle`: チャネルごとの上書き。
-   `channels.msteams.teams..channels..requireMention`: チャネルごとの上書き。
-   `channels.msteams.teams..channels..tools`: チャネルごとのツールポリシー上書き (`allow`/`deny`/`alsoAllow`)。
-   `channels.msteams.teams..channels..toolsBySender`: チャネルごとの送信者ごとのツールポリシー上書き (`"*"` ワイルドカードサポート)。
-   `toolsBySender` キーは明示的なプレフィックスを使用する必要があります: `id:`、`e164:`、`username:`、`name:` (レガシーのプレフィックスなしキーは依然として `id:` のみにマッピングされます)。
-   `channels.msteams.sharePointSiteId`: グループチャット/チャネルでのファイルアップロード用の SharePoint サイト ID ([グループチャットでのファイル送信](#sending-files-in-group-chats) を参照)。

## ルーティング & セッション

-   セッションキーは標準のエージェント形式に従います ([/concepts/session](../concepts/session.md) を参照):
    -   ダイレクトメッセージはメインセッションを共有します (`agent::`)。
    -   チャネル/グループメッセージは会話 ID を使用します:
        -   `agent::msteams:channel:`
        -   `agent::msteams:group:`

## 返信スタイル: スレッド vs 投稿

Teams は最近、同じ基盤データモデル上で2つのチャネル UI スタイルを導入しました:

| スタイル | 説明 | 推奨 `replyStyle` |
| --- | --- | --- |
| **投稿** (クラシック) | メッセージはカードとして表示され、その下にスレッド返信が表示されます | `thread` (デフォルト) |
| **スレッド** (Slack 風) | メッセージが直線的に流れ、Slack に似ています | `top-level` |

**問題点:** Teams API はチャネルがどの UI スタイルを使用しているかを公開しません。誤った `replyStyle` を使用すると:

-   `thread` をスレッドスタイルのチャネルで使用 → 返信が不自然にネストして表示されます。
-   `top-level` を投稿スタイルのチャネルで使用 → 返信がスレッド内ではなく、別々のトップレベル投稿として表示されます。

**解決策:** チャネルの設定方法に基づいて、チャネルごとに `replyStyle` を設定します:

```json
{
  "msteams": {
    "replyStyle": "thread",
    "teams": {
      "19:abc...@thread.tacv2": {
        "channels": {
          "19:xyz...@thread.tacv2": {
            "replyStyle": "top-level"
          }
        }
      }
    }
  }
}
```

## 添付ファイル & 画像

**現在の制限:**

-   **DM:** 画像とファイル添付は Teams ボットファイル API 経由で動作します。
-   **チャネル/グループ:** 添付ファイルは M365 ストレージ (SharePoint/OneDrive) に存在します。Webhook ペイロードには実際のファイルバイトではなく HTML スタブのみが含まれます。チャネル添付ファイルをダウンロードするには **Graph API 権限が必要です**。

Graph 権限がない場合、画像を含むチャネルメッセージはテキストのみとして受信されます (画像コンテンツはボットからアクセスできません)。デフォルトでは、OpenClaw は Microsoft/Teams ホスト名からのメディアのみをダウンロードします。`channels.msteams.mediaAllowHosts` で上書きします (任意のホストを許可するには `["*"]` を使用)。Authorization ヘッダーは、`channels.msteams.mediaAuthAllowHosts` 内のホストに対してのみ添付されます (デフォルトは Graph + Bot Framework ホスト)。このリストは厳密に保ってください (マルチテナントサフィックスは避けてください)。

## グループチャットでのファイル送信

ボットは FileConsentCard フロー (組み込み) を使用して DM でファイルを送信できます。ただし、**グループチャット/チャネルでのファイル送信** には追加のセットアップが必要です:

| コンテキスト | ファイルの送信方法 | 必要なセットアップ |
| --- | --- | --- |
| **DM** | FileConsentCard → ユーザーが承諾 → ボットがアップロード | そのまま動作 |
| **グループチャット/チャネル** | SharePoint にアップロード → 共有リンク | `sharePointSiteId` + Graph 権限が必要 |
| **画像 (任意のコンテキスト)** | Base64 エンコードされたインライン | そのまま動作 |

### グループチャットが SharePoint を必要とする理由

ボットには個人用の OneDrive ドライブがありません (アプリケーション ID では `/me/drive` Graph API エンドポイントは動作しません)。グループチャット/チャネルでファイルを送信するには