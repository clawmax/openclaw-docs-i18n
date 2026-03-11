

  メディアとデバイス

  
# カメラキャプチャ

OpenClawはエージェントワークフローでの**カメラキャプチャ**をサポートしています：

-   **iOSノード**（Gateway経由でペアリング）：`node.invoke`経由で**写真**（`jpg`）または**短い動画クリップ**（`mp4`、オーディオオプション付き）をキャプチャ。
-   **Androidノード**（Gateway経由でペアリング）：`node.invoke`経由で**写真**（`jpg`）または**短い動画クリップ**（`mp4`、オーディオオプション付き）をキャプチャ。
-   **macOSアプリ**（Gateway経由のノード）：`node.invoke`経由で**写真**（`jpg`）または**短い動画クリップ**（`mp4`、オーディオオプション付き）をキャプチャ。

すべてのカメラアクセスは**ユーザー制御の設定**によって制限されます。

## iOSノード

### ユーザー設定（デフォルトオン）

-   iOS設定タブ → **カメラ** → **カメラを許可**（`camera.enabled`）
    -   デフォルト：**オン**（キーがない場合は有効として扱われます）。
    -   オフの場合：`camera.*`コマンドは`CAMERA_DISABLED`を返します。

### コマンド（Gateway node.invoke経由）

-   `camera.list`
    -   レスポンスペイロード：
        -   `devices`: `{ id, name, position, deviceType }`の配列
-   `camera.snap`
    -   パラメータ：
        -   `facing`: `front|back`（デフォルト：`front`）
        -   `maxWidth`: 数値（オプション；iOSノードでのデフォルトは`1600`）
        -   `quality`: `0..1`（オプション；デフォルト`0.9`）
        -   `format`: 現在`jpg`
        -   `delayMs`: 数値（オプション；デフォルト`0`）
        -   `deviceId`: 文字列（オプション；`camera.list`から）
    -   レスポンスペイロード：
        -   `format: "jpg"`
        -   `base64: "<...>"`
        -   `width`, `height`
    -   ペイロードガード：base64ペイロードを5 MB未満に保つために写真は再圧縮されます。
-   `camera.clip`
    -   パラメータ：
        -   `facing`: `front|back`（デフォルト：`front`）
        -   `durationMs`: 数値（デフォルト`3000`、最大`60000`に制限）
        -   `includeAudio`: ブール値（デフォルト`true`）
        -   `format`: 現在`mp4`
        -   `deviceId`: 文字列（オプション；`camera.list`から）
    -   レスポンスペイロード：
        -   `format: "mp4"`
        -   `base64: "<...>"`
        -   `durationMs`
        -   `hasAudio`

### フォアグラウンド要件

`canvas.*`と同様に、iOSノードは`camera.*`コマンドを**フォアグラウンド**でのみ許可します。バックグラウンドでの呼び出しは`NODE_BACKGROUND_UNAVAILABLE`を返します。

### CLIヘルパー（一時ファイル + MEDIA）

アタッチメントを取得する最も簡単な方法はCLIヘルパーを使用することです。これはデコードされたメディアを一時ファイルに書き込み、`MEDIA:<パス>`を出力します。例：

```bash
openclaw nodes camera snap --node <id>               # デフォルト：前面と背面の両方（2行のMEDIA出力）
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

注意：

-   `nodes camera snap`は、エージェントに両方の視点を提供するために、デフォルトで**両方**の向き（前面と背面）をキャプチャします。
-   出力ファイルは一時的なものです（OSの一時ディレクトリ内）。独自のラッパーを作成しない限り。

## Androidノード

### Androidユーザー設定（デフォルトオン）

-   Android設定シート → **カメラ** → **カメラを許可**（`camera.enabled`）
    -   デフォルト：**オン**（キーがない場合は有効として扱われます）。
    -   オフの場合：`camera.*`コマンドは`CAMERA_DISABLED`を返します。

### 権限

-   Androidは実行時権限を必要とします：
    -   `camera.snap`と`camera.clip`の両方に`CAMERA`権限。
    -   `includeAudio=true`の場合の`camera.clip`に`RECORD_AUDIO`権限。

権限がない場合、可能であればアプリがプロンプトを表示します。拒否された場合、`camera.*`リクエストは`*_PERMISSION_REQUIRED`エラーで失敗します。

### Androidフォアグラウンド要件

`canvas.*`と同様に、Androidノードは`camera.*`コマンドを**フォアグラウンド**でのみ許可します。バックグラウンドでの呼び出しは`NODE_BACKGROUND_UNAVAILABLE`を返します。

### Androidコマンド（Gateway node.invoke経由）

-   `camera.list`
    -   レスポンスペイロード：
        -   `devices`: `{ id, name, position, deviceType }`の配列

### ペイロードガード

写真はbase64ペイロードを5 MB未満に保つために再圧縮されます。

## macOSアプリ

### ユーザー設定（デフォルトオフ）

macOSコンパニオンアプリはチェックボックスを公開します：

-   **設定 → 一般 → カメラを許可**（`openclaw.cameraEnabled`）
    -   デフォルト：**オフ**
    -   オフの場合：カメラリクエストは「ユーザーによってカメラが無効化されています」を返します。

### CLIヘルパー（node invoke）

メインの`openclaw` CLIを使用して、macOSノードでカメラコマンドを呼び出します。例：

```bash
openclaw nodes camera list --node <id>            # カメラIDをリスト表示
openclaw nodes camera snap --node <id>            # MEDIA:<パス>を出力
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # MEDIA:<パス>を出力
openclaw nodes camera clip --node <id> --duration-ms 3000      # MEDIA:<パス>を出力（従来のフラグ）
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

注意：

-   `openclaw nodes camera snap`は、上書きされない限り、デフォルトで`maxWidth=1600`を使用します。
-   macOSでは、`camera.snap`はウォームアップ/露出が安定した後、`delayMs`（デフォルト2000ms）待機してからキャプチャします。
-   写真ペイロードはbase64を5 MB未満に保つために再圧縮されます。

## 安全性と実用的な制限

-   カメラとマイクへのアクセスは、通常のOS権限プロンプトをトリガーします（また、Info.plistでの使用説明文字列が必要です）。
-   動画クリップは、過大なノードペイロード（base64オーバーヘッド + メッセージ制限）を避けるために制限されています（現在`<= 60s`）。

## macOS画面動画（OSレベル）

*画面*動画（カメラではない）には、macOSコンパニオンを使用します：

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # MEDIA:<パス>を出力
```

注意：

-   macOSの**画面収録**権限（TCC）が必要です。

[オーディオと音声メモ](./audio.md)[トークモード](./talk.md)