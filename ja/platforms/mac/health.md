

  macOSコンパニオンアプリ

  
# ヘルスチェック

メニューバーアプリからリンクされたチャネルが健全かどうかを確認する方法。

## メニューバー

-   ステータスドットがBaileysの健全性を反映:
    -   緑: リンク済み + 最近ソケットが開かれました。
    -   オレンジ: 接続中/再試行中。
    -   赤: ログアウト済みまたはプローブ失敗。
-   2行目には「linked · auth 12m」と表示されるか、失敗理由が表示されます。
-   「Run Health Check」メニュー項目はオンデマンドプローブをトリガーします。

## 設定

-   一般タブにヘルスカードが追加され、以下を表示: リンク済み認証の経過時間、セッションストアのパス/カウント、最終チェック時刻、最終エラー/ステータスコード、および「Run Health Check」/「Reveal Logs」ボタン。
-   キャッシュされたスナップショットを使用するため、UIは瞬時に読み込まれ、オフライン時も適切にフォールバックします。
-   **チャネルタブ**では、WhatsApp/Telegramのチャネルステータスとコントロール（ログインQR、ログアウト、プローブ、最終切断/エラー）が表示されます。

## プローブの仕組み

-   アプリは約60秒ごとおよびオンデマンドで、`ShellExecutor`経由で`openclaw health --json`を実行します。このプローブは認証情報を読み込み、メッセージを送信せずにステータスを報告します。
-   最後の正常なスナップショットと最後のエラーを別々にキャッシュしてちらつきを防ぎ、それぞれのタイムスタンプを表示します。

## 不明な点がある場合

-   [Gateway health](../../gateway/health.md) (`openclaw status`, `openclaw status --deep`, `openclaw health --json`) のCLIフローを引き続き使用でき、`/tmp/openclaw/openclaw-*.log`を`web-heartbeat` / `web-reconnect`のためにtailできます。

[Gateway Lifecycle](./child-process.md)[Menu Bar Icon](./icon.md)