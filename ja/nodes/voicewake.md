

  メディアとデバイス

  
# Voice Wake

OpenClaw は**ウェイクワードを単一のグローバルリスト**として扱い、**ゲートウェイ**が所有します。

-   **ノードごとのカスタムウェイクワードはありません**。
-   **任意のノード/アプリ UI がリストを編集可能**です。変更はゲートウェイによって永続化され、全員にブロードキャストされます。
-   macOS と iOS は、ローカルの **Voice Wake 有効/無効トグル**を保持します（ローカル UX と権限が異なります）。
-   Android は現在、Voice Wake をオフに保ち、Voice タブで手動マイクフローを使用します。

## ストレージ (ゲートウェイホスト)

ウェイクワードは、ゲートウェイマシンの以下の場所に保存されます:

-   `~/.openclaw/settings/voicewake.json`

構造:

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## プロトコル

### メソッド

-   `voicewake.get` → `{ triggers: string[] }`
-   `voicewake.set` パラメータ `{ triggers: string[] }` で → `{ triggers: string[] }`

注記:

-   トリガーは正規化されます（トリミング、空の削除）。空のリストはデフォルトにフォールバックします。
-   安全性のため制限が適用されます（数/長さの上限）。

### イベント

-   `voicewake.changed` ペイロード `{ triggers: string[] }`

受信者:

-   すべての WebSocket クライアント (macOS アプリ、WebChat など)
-   すべての接続されたノード (iOS/Android)。また、ノード接続時には初期の「現在の状態」プッシュとしても送信されます。

## クライアントの動作

### macOS アプリ

-   グローバルリストを使用して `VoiceWakeRuntime` トリガーを制御します。
-   Voice Wake 設定で「トリガーワード」を編集すると、`voicewake.set` が呼び出され、その後ブロードキャストに依存して他のクライアントを同期状態に保ちます。

### iOS ノード

-   グローバルリストを `VoiceWakeManager` のトリガー検出に使用します。
-   設定でウェイクワードを編集すると、`voicewake.set` が呼び出され（ゲートウェイ WS 経由）、ローカルのウェイクワード検出も応答性を保ちます。

### Android ノード

-   Voice Wake は現在、Android ランタイム/設定で無効になっています。
-   Android の音声機能は、ウェイクワードトリガーの代わりに、Voice タブでの手動マイクキャプチャを使用します。

[トークモード](./talk.md)[位置情報コマンド](./location-command.md)