

  macOS コンパニオンアプリ

  
# WebChat

macOSメニューバーアプリは、WebChat UIをネイティブのSwiftUIビューとして組み込んでいます。ゲートウェイに接続し、選択されたエージェントの**メインセッション**をデフォルトとします（他のセッション用のセッションスイッチャー付き）。

-   **ローカルモード**: ローカルのゲートウェイWebSocketに直接接続します。
-   **リモートモード**: SSH経由でゲートウェイの制御ポートを転送し、そのトンネルをデータプレーンとして使用します。

## 起動とデバッグ

-   手動: Lobsterメニュー → 「チャットを開く」。
-   テスト用の自動起動:
    
    コピー
    
    ```bash
    dist/OpenClaw.app/Contents/MacOS/OpenClaw --webchat
    ```
    
-   ログ: `./scripts/clawlog.sh` (サブシステム `ai.openclaw`, カテゴリ `WebChatSwiftUI`)。

## 接続構成

-   データプレーン: ゲートウェイWSメソッド `chat.history`, `chat.send`, `chat.abort`, `chat.inject` およびイベント `chat`, `agent`, `presence`, `tick`, `health`。
-   セッション: プライマリセッション（`main`、またはスコープがグローバルの場合は`global`）がデフォルトです。UIはセッション間を切り替えることができます。
-   オンボーディングは専用のセッションを使用し、初回セットアップを分離します。

## セキュリティ面

-   リモートモードでは、ゲートウェイWebSocket制御ポートのみがSSH経由で転送されます。

## 既知の制限事項

-   UIはチャットセッション用に最適化されています（完全なブラウザサンドボックスではありません）。

[Voice Overlay](./voice-overlay.md)[Canvas](./canvas.md)

---