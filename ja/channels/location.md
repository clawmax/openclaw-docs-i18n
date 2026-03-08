

  設定

  
# チャネル位置情報パース

OpenClawは、チャットチャネルから共有された位置情報を以下のように正規化します：

-   人間が読めるテキストとして受信本文に追加され、
-   自動返信コンテキストペイロード内の構造化フィールドに格納されます。

現在サポートされているもの：

-   **Telegram** (位置ピン + 施設情報 + ライブ位置情報)
-   **WhatsApp** (locationMessage + liveLocationMessage)
-   **Matrix** (`m.location` と `geo_uri`)

## テキストフォーマット

位置情報は、括弧なしのわかりやすい行としてレンダリングされます：

-   ピン：
    -   `📍 48.858844, 2.294351 ±12m`
-   名前付き場所：
    -   `📍 エッフェル塔 — シャン・ド・マルス, パリ (48.858844, 2.294351 ±12m)`
-   ライブ共有：
    -   `🛰 ライブ位置情報: 48.858844, 2.294351 ±12m`

チャネルにキャプション/コメントが含まれている場合は、次の行に追加されます：

```
📍 48.858844, 2.294351 ±12m
ここで待ち合わせ
```

## コンテキストフィールド

位置情報が存在する場合、以下のフィールドが `ctx` に追加されます：

-   `LocationLat` (数値)
-   `LocationLon` (数値)
-   `LocationAccuracy` (数値、メートル単位; オプション)
-   `LocationName` (文字列; オプション)
-   `LocationAddress` (文字列; オプション)
-   `LocationSource` (`pin | place | live`)
-   `LocationIsLive` (ブール値)

## チャネルごとの注意点

-   **Telegram**: 施設情報は `LocationName`/`LocationAddress` にマッピングされます。ライブ位置情報は `live_period` を使用します。
-   **WhatsApp**: `locationMessage.comment` と `liveLocationMessage.caption` はキャプション行として追加されます。
-   **Matrix**: `geo_uri` はピン位置としてパースされます。高度は無視され、`LocationIsLive` は常に false です。

[チャネルルーティング](./channel-routing.md)[チャネルトラブルシューティング](./troubleshooting.md)