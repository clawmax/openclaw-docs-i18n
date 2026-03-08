

  組み込みツール

  
# Reactions

チャネル間で共通するリアクションのセマンティクス:

-   リアクションを追加する場合、`emoji` は必須です。
-   `emoji=""` は、サポートされている場合、ボットのリアクションを削除します。
-   `remove: true` は、サポートされている場合、指定された絵文字を削除します (`emoji` が必要です)。

チャネルごとの注意点:

-   **Discord/Slack**: `emoji` を空にすると、メッセージ上のボットのすべてのリアクションを削除します。`remove: true` はその絵文字のみを削除します。
-   **Google Chat**: `emoji` を空にすると、メッセージ上のアプリのリアクションを削除します。`remove: true` はその絵文字のみを削除します。
-   **Telegram**: `emoji` を空にすると、ボットのリアクションを削除します。`remove: true` もリアクションを削除しますが、ツールの検証のために空でない `emoji` が必要です。
-   **WhatsApp**: `emoji` を空にすると、ボットのリアクションを削除します。`remove: true` は空の絵文字にマッピングされます (それでも `emoji` は必要です)。
-   **Zalo Personal (`zalouser`)**: 空でない `emoji` が必要です。`remove: true` はその特定の絵文字リアクションを削除します。
-   **Signal**: `channels.signal.reactionNotifications` が有効な場合、受信したリアクション通知はシステムイベントを発行します。

[ツールループ検出](./loop-detection.md)[思考レベル](./thinking.md)

---