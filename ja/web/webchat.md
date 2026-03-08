title: "OpenClaw AI Gateway統合のためのWebChatガイド"
description: "OpenClaw AI Gateway用のネイティブWebChat UIのセットアップと使用方法を学びます。WebSocket接続の設定、チャット履歴の管理、リモートアクセスの理解について説明します。"
keywords: ["ウェブチャット", "openclawゲートウェイ", "websocketチャット", "チャットui", "ゲートウェイ設定", "チャット履歴", "コントロールui", "リモートアクセス"]
---

  ウェブインターフェース

  
# WebChat

ステータス: macOS/iOS SwiftUIチャットUIは、ゲートウェイWebSocketに直接通信します。

## 概要

-   ゲートウェイ用のネイティブチャットUIです（埋め込みブラウザやローカルの静的サーバーは不要）。
-   他のチャネルと同じセッションとルーティングルールを使用します。
-   決定論的ルーティング: 返信は常にWebChatに戻ります。

## クイックスタート

1.  ゲートウェイを起動します。
2.  WebChat UI（macOS/iOSアプリ）またはControl UIのチャットタブを開きます。
3.  ゲートウェイ認証が設定されていることを確認します（デフォルトではループバックでも必須です）。

## 動作の仕組み

-   UIはゲートウェイWebSocketに接続し、`chat.history`、`chat.send`、`chat.inject`を使用します。
-   `chat.history`は安定性のために制限されています: ゲートウェイは長いテキストフィールドを切り詰めたり、重いメタデータを省略したり、サイズ超過のエントリを`[chat.history omitted: message too large]`で置き換えたりする場合があります。
-   `chat.inject`はアシスタントのメモを直接トランスクリプトに追加し、UIにブロードキャストします（エージェントの実行はありません）。
-   中止された実行では、部分的に出力されたアシスタントの応答がUIに表示されたままになることがあります。
-   ゲートウェイは、バッファリングされた出力が存在する場合、中止された部分的なアシスタントテキストをトランスクリプト履歴に永続化し、それらのエントリを中止メタデータでマークします。
-   履歴は常にゲートウェイから取得されます（ローカルファイルの監視はありません）。
-   ゲートウェイに到達できない場合、WebChatは読み取り専用になります。

## Control UI エージェントツールパネル

-   Control UIの`/agents`ツールパネルは、`tools.catalog`を介してランタイムカタログを取得し、各ツールを`core`または`plugin:`としてラベル付けします（オプションのプラグインツールには`optional`も付加）。
-   `tools.catalog`が利用できない場合、パネルは組み込みの静的リストにフォールバックします。
-   パネルはプロファイルとオーバーライド設定を編集しますが、有効なランタイムアクセスは依然としてポリシーの優先順位（`allow`/`deny`、エージェントごとおよびプロバイダー/チャネルオーバーライド）に従います。

## リモート使用

-   リモートモードでは、ゲートウェイWebSocketをSSH/Tailscale経由でトンネリングします。
-   別途WebChatサーバーを実行する必要はありません。

## 設定リファレンス (WebChat)

完全な設定: [設定](../gateway/configuration.md) チャネルオプション:

-   専用の`webchat.*`ブロックはありません。WebChatは以下のゲートウェイエンドポイント + 認証設定を使用します。

関連するグローバルオプション:

-   `gateway.port`, `gateway.bind`: WebSocketホスト/ポート。
-   `gateway.auth.mode`, `gateway.auth.token`, `gateway.auth.password`: WebSocket認証（トークン/パスワード）。
-   `gateway.auth.mode: "trusted-proxy"`: ブラウザクライアント用のリバースプロキシ認証（[信頼済みプロキシ認証](../gateway/trusted-proxy-auth.md)を参照）。
-   `gateway.remote.url`, `gateway.remote.token`, `gateway.remote.password`: リモートゲートウェイターゲット。
-   `session.*`: セッションストレージとメインキーのデフォルト。

[ダッシュボード](./dashboard.md)[TUI](./tui.md)

---