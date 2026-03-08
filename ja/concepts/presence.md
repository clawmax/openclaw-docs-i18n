title: "マルチエージェントシステム監視のためのOpenClaw Presenceコンセプト"
description: "OpenClaw presenceについて学びましょう。これはGatewayと接続されたクライアントの軽量なビューです。そのフィールド、プロデューサー、マージルール、そしてどのようにInstancesタブを動かしているかを理解します。"
keywords: ["openclaw presence", "マルチエージェント監視", "ゲートウェイクライアント", "instanceid", "system-presence", "websocket接続", "presence ttl", "macosインスタンストラブ"]
---

  マルチエージェント

  
# Presence

OpenClawの「presence」は、以下の軽量でベストエフォートなビューです：

-   **Gateway**自体、および
-   **Gatewayに接続されたクライアント**（macアプリ、WebChat、CLIなど）

Presenceは主に、macOSアプリの**Instances**タブをレンダリングし、オペレーターに迅速な可視性を提供するために使用されます。

## Presenceフィールド（表示される内容）

Presenceエントリは、以下のようなフィールドを持つ構造化オブジェクトです：

-   `instanceId`（オプションだが強く推奨）：安定したクライアント識別子（通常は`connect.client.instanceId`）
-   `host`：人間が読みやすいホスト名
-   `ip`：ベストエフォートのIPアドレス
-   `version`：クライアントバージョン文字列
-   `deviceFamily` / `modelIdentifier`：ハードウェアのヒント
-   `mode`：`ui`, `webchat`, `cli`, `backend`, `probe`, `test`, `node`, …
-   `lastInputSeconds`：「最後のユーザー入力からの経過秒数」（既知の場合）
-   `reason`：`self`, `connect`, `node-connected`, `periodic`, …
-   `ts`：最終更新タイムスタンプ（エポックからのミリ秒）

## プロデューサー（presenceの発生源）

Presenceエントリは複数のソースによって生成され、**マージ**されます。

### 1) Gateway自己エントリ

Gatewayは起動時に常に「self」エントリを生成するため、クライアントが接続する前でもUIにゲートウェイホストが表示されます。

### 2) WebSocket接続

すべてのWSクライアントは`connect`リクエストから始まります。ハンドシェイクが成功すると、Gatewayはその接続のpresenceエントリをアップサートします。

#### ワンオフCLIコマンドが表示されない理由

CLIはしばしば短いワンオフコマンドのために接続します。Instancesリストがスパム化されるのを防ぐため、`client.mode === "cli"`はpresenceエントリに**変換されません**。

### 3) system-eventビーコン

クライアントは`system-event`メソッドを介して、より詳細な定期的なビーコンを送信できます。macアプリはこれを使用して、ホスト名、IP、`lastInputSeconds`を報告します。

### 4) ノード接続（role: node）

ノードが`role: node`でGateway WebSocketを介して接続すると、Gatewayはそのノードのpresenceエントリをアップサートします（他のWSクライアントと同じフロー）。

## マージ + 重複排除ルール（instanceIdが重要な理由）

Presenceエントリは単一のインメモリマップに保存されます：

-   エントリは**presenceキー**によってキー付けされます。
-   最良のキーは、再起動後も存続する安定した`instanceId`（`connect.client.instanceId`から）です。
-   キーは大文字と小文字を区別しません。

安定した`instanceId`なしでクライアントが再接続すると、**重複**行として表示される可能性があります。

## TTLとサイズ制限

Presenceは意図的に一時的なものです：

-   **TTL:** 5分以上経過したエントリは削除されます
-   **最大エントリ数:** 200（最も古いものから順に削除）

これによりリストは新鮮に保たれ、無制限なメモリ増加を防ぎます。

## リモート/トンネルに関する注意点（ループバックIP）

クライアントがSSHトンネル/ローカルポートフォワードを介して接続すると、Gatewayはリモートアドレスを`127.0.0.1`として認識する可能性があります。クライアントから報告された適切なIPを上書きしないようにするため、ループバックリモートアドレスは無視されます。

## コンシューマー

### macOS Instancesタブ

macOSアプリは`system-presence`の出力をレンダリングし、最終更新からの経過時間に基づいて小さなステータスインジケータ（アクティブ/アイドル/古い）を適用します。

## デバッグのヒント

-   生のリストを確認するには、Gatewayに対して`system-presence`を呼び出します。
-   重複が見られる場合：
    -   クライアントがハンドシェイクで安定した`client.instanceId`を送信していることを確認してください
    -   定期的なビーコンが同じ`instanceId`を使用していることを確認してください
    -   接続から派生したエントリに`instanceId`が欠落していないか確認してください（重複は予想されることです）

[マルチエージェントルーティング](./multi-agent.md)[メッセージ](./messages.md)