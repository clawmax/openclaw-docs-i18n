

  設定と運用

  
# ヘルスチェック

推測せずにチャネル接続を確認するための短いガイドです。

## クイックチェック

-   `openclaw status` — ローカルサマリー: ゲートウェイ到達性/モード、更新ヒント、リンク済みチャネル認証の経過時間、セッション数 + 最近のアクティビティ。
-   `openclaw status --all` — 完全なローカル診断 (読み取り専用、カラー表示、デバッグ用に貼り付けても安全)。
-   `openclaw status --deep` — 実行中のゲートウェイもプローブします (サポートされている場合はチャネルごとのプローブ)。
-   `openclaw health --json` — 実行中のゲートウェイに完全なヘルススナップショットを要求します (WSのみ; 直接のBaileysソケットは使用しません)。
-   WhatsApp/WebChatで `/status` を単独のメッセージとして送信すると、エージェントを起動せずにステータス応答を取得できます。
-   ログ: `/tmp/openclaw/openclaw-*.log` を tail し、`web-heartbeat`、`web-reconnect`、`web-auto-reply`、`web-inbound` でフィルタリングします。

## 詳細診断

-   ディスク上の認証情報: `ls -l ~/.openclaw/credentials/whatsapp//creds.json` (mtime は最近であるべきです)。
-   セッションストア: `ls -l ~/.openclaw/agents//sessions/sessions.json` (パスは設定で上書き可能)。数と最近の受信者は `status` コマンドで表示されます。
-   再リンクフロー: ログにステータスコード 409–515 または `loggedOut` が表示されたら、`openclaw channels logout && openclaw channels login --verbose` を実行します。(注: QRログインフローは、ペアリング後のステータス515に対して一度自動再起動します。)

## 問題発生時

-   `logged out` またはステータス 409–515 → `openclaw channels logout` の後、`openclaw channels login` で再リンクします。
-   ゲートウェイに到達できない → 起動します: `openclaw gateway --port 18789` (ポートがビジーの場合は `--force` を使用)。
-   受信メッセージがない → リンク済みの電話がオンラインであること、送信者が許可されていることを確認します (`channels.whatsapp.allowFrom`)。グループチャットの場合は、許可リストとメンションルールが一致していることを確認します (`channels.whatsapp.groups`, `agents.list[].groupChat.mentionPatterns`)。

## 専用の「health」コマンド

`openclaw health --json` は、実行中のゲートウェイにそのヘルススナップショットを要求します (CLIから直接チャネルソケットを使用しません)。利用可能な場合はリンク済みの認証情報/認証経過時間、チャネルごとのプローブサマリー、セッションストアのサマリー、およびプローブ所要時間を報告します。ゲートウェイに到達できない場合、またはプローブが失敗/タイムアウトした場合は非ゼロで終了します。デフォルトの10秒を上書きするには `--timeout ` を使用します。

[信頼されたプロキシ認証](./trusted-proxy-auth.md)[ハートビート](./heartbeat.md)