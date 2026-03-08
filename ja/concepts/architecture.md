

  基礎

  
# Gateway アーキテクチャ

最終更新日: 2026-01-22

## 概要

-   単一の長寿命な **Gateway** が、すべてのメッセージングサーフェス (Baileys 経由の WhatsApp、grammY 経由の Telegram、Slack、Discord、Signal、iMessage、WebChat) を所有します。
-   コントロールプレーンクライアント (macOS アプリ、CLI、Web UI、自動化) は、設定されたバインドホスト (デフォルト `127.0.0.1:18789`) 上の **WebSocket** を介して Gateway に接続します。
-   **ノード** (macOS/iOS/Android/ヘッドレス) も **WebSocket** を介して接続しますが、明示的な caps/commands と共に `role: node` を宣言します。
-   ホストごとに 1 つの Gateway。これは WhatsApp セッションを開く唯一の場所です。
-   **キャンバスホスト** は、Gateway HTTP サーバーによって以下のパスで提供されます:
    -   `/__openclaw__/canvas/` (エージェント編集可能な HTML/CSS/JS)
    -   `/__openclaw__/a2ui/` (A2UI ホスト) Gateway と同じポート (デフォルト `18789`) を使用します。

## コンポーネントとフロー

### Gateway (デーモン)

-   プロバイダー接続を維持します。
-   型付き WS API (リクエスト、レスポンス、サーバープッシュイベント) を公開します。
-   受信フレームを JSON Schema に対して検証します。
-   `agent`、`chat`、`presence`、`health`、`heartbeat`、`cron` などのイベントを発行します。

### クライアント (mac アプリ / CLI / Web 管理画面)

-   クライアントごとに 1 つの WS 接続。
-   リクエスト (`health`、`status`、`send`、`agent`、`system-presence`) を送信します。
-   イベント (`tick`、`agent`、`presence`、`shutdown`) を購読します。

### ノード (macOS / iOS / Android / ヘッドレス)

-   `role: node` を指定して **同じ WS サーバー** に接続します。
-   `connect` でデバイス ID を提供します。ペアリングは **デバイスベース** (ロール `node`) であり、承認はデバイスペアリングストアに保存されます。
-   `canvas.*`、`camera.*`、`screen.record`、`location.get` などのコマンドを公開します。

プロトコルの詳細:

-   [Gateway プロトコル](../gateway/protocol.md)

### WebChat

-   チャット履歴と送信のために Gateway WS API を使用する静的 UI。
-   リモート設定では、他のクライアントと同じ SSH/Tailscale トンネルを介して接続します。

## 接続ライフサイクル (単一クライアント)

## ワイヤプロトコル (概要)

-   トランスポート: WebSocket、JSON ペイロードを持つテキストフレーム。
-   最初のフレーム **は必ず** `connect` でなければなりません。
-   ハンドシェイク後:
    -   リクエスト: `{type:"req", id, method, params}` → `{type:"res", id, ok, payload|error}`
    -   イベント: `{type:"event", event, payload, seq?, stateVersion?}`
-   `OPENCLAW_GATEWAY_TOKEN` (または `--token`) が設定されている場合、`connect.params.auth.token` が一致しなければソケットは閉じられます。
-   べき等性キーは、副作用のあるメソッド (`send`、`agent`) を安全に再試行するために必要です。サーバーは短命な重複排除キャッシュを保持します。
-   ノードは `connect` に `role: "node"` と caps/commands/permissions を含める必要があります。

## ペアリング + ローカル信頼

-   すべての WS クライアント (オペレーター + ノード) は、`connect` 時に **デバイス ID** を含めます。
-   新しいデバイス ID はペアリング承認を必要とします。Gateway は、その後の接続のために **デバイストークン** を発行します。
-   **ローカル** 接続 (ループバックまたは Gateway ホスト自身の tailnet アドレス) は、同じホストでの UX をスムーズにするために自動承認されることがあります。
-   すべての接続は `connect.challenge` ノンスに署名する必要があります。
-   署名ペイロード `v3` は `platform` + `deviceFamily` もバインドします。Gateway は再接続時にペアリングされたメタデータを固定し、メタデータ変更時にはペアリングの再設定を要求します。
-   **非ローカル** 接続は、依然として明示的な承認を必要とします。
-   Gateway 認証 (`gateway.auth.*`) は、ローカルまたはリモートを問わず **すべての** 接続に適用されます。

詳細: [Gateway プロトコル](../gateway/protocol.md), [ペアリング](../channels/pairing.md), [セキュリティ](../gateway/security.md)。

## プロトコルの型付けとコード生成

-   TypeBox スキーマがプロトコルを定義します。
-   JSON Schema はそれらのスキーマから生成されます。
-   Swift モデルは JSON Schema から生成されます。

## リモートアクセス

-   推奨: Tailscale または VPN。
-   代替: SSH トンネル
    
    コピー
    
    ```bash
    ssh -N -L 18789:127.0.0.1:18789 user@host
    ```
    
-   同じハンドシェイク + 認証トークンがトンネル経由でも適用されます。
-   TLS + オプションのピン留めは、リモート設定での WS に対して有効にできます。

## 運用スナップショット

-   起動: `openclaw gateway` (フォアグラウンド、ログは stdout へ)。
-   ヘルスチェック: WS 経由の `health` (`hello-ok` にも含まれます)。
-   監視: 自動再起動のための launchd/systemd。

## 不変条件

-   正確に 1 つの Gateway がホストごとに単一の Baileys セッションを制御します。
-   ハンドシェイクは必須です。JSON でない、または最初のフレームが connect でないものは強制クローズされます。
-   イベントは再生されません。クライアントはギャップ発生時にリフレッシュする必要があります。

[Pi 統合アーキテクチャ](../pi.md)[エージェントランタイム](./agent.md)