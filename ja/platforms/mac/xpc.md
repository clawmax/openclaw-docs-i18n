title: "OpenClaw macOS XPC IPC およびコンパニオンアプリ アーキテクチャ"
description: "OpenClaw macOS コンパニオンプアプリが、安全な IPC、TCC 権限、および PeekabooBridge を介した UI 自動化のために XPC と Unix ソケットをどのように使用するかを学びます。"
keywords: ["macos xpc", "openclaw mac", "ipc unix ソケット", "tcc 権限", "peekaboobridge", "macos コンパニオン アプリ", "ノード ホスト サービス", "ui 自動化"]
---

  macOS コンパニオンアプリ

  
# macOS IPC

**現在のモデル:** ローカル Unix ソケットが **ノード ホスト サービス** を **macOS アプリ** に接続し、実行承認と `system.run` を処理します。 `openclaw-mac` デバッグ CLI が検出/接続チェック用に存在します。エージェントのアクションは依然として Gateway WebSocket と `node.invoke` を経由します。UI 自動化には PeekabooBridge を使用します。

## 目標

-   TCC 関連の作業（通知、画面録画、マイク、音声認識、AppleScript）をすべて担当する単一の GUI アプリインスタンス。
-   自動化のための小さなインターフェース: Gateway + ノードコマンド、および UI 自動化のための PeekabooBridge。
-   予測可能な権限: 常に同じ署名付きバンドル ID、launchd によって起動されるため、TCC の許可が保持される。

## 仕組み

### Gateway + ノード トランスポート

-   アプリは Gateway（ローカルモード）を実行し、ノードとして接続します。
-   エージェントのアクションは `node.invoke` を介して実行されます（例: `system.run`, `system.notify`, `canvas.*`）。

### ノード サービス + アプリ間 IPC

-   ヘッドレスのノード ホスト サービスが Gateway WebSocket に接続します。
-   `system.run` リクエストは、ローカル Unix ソケットを介して macOS アプリに転送されます。
-   アプリは UI コンテキストで実行を行い、必要に応じてプロンプトを表示し、出力を返します。

ダイアグラム (SCI):

```
エージェント -> Gateway -> ノード サービス (WS)
                      |  IPC (UDS + トークン + HMAC + TTL)
                      v
                  Mac アプリ (UI + TCC + system.run)
```

### PeekabooBridge (UI 自動化)

-   UI 自動化には、`bridge.sock` という別の UNIX ソケットと PeekabooBridge JSON プロトコルを使用します。
-   ホストの優先順位（クライアント側）: Peekaboo.app → Claude.app → OpenClaw.app → ローカル実行。
-   セキュリティ: ブリッジホストには許可された TeamID が必要です。DEBUG 専用の同一 UID エスケープハッチは `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (Peekaboo の慣例) によって保護されています。
-   詳細は [PeekabooBridge の使用法](./peekaboo.md) を参照してください。

## 運用フロー

-   再起動/リビルド: `SIGN_IDENTITY="Apple Development: <開発者名> ()" scripts/restart-mac.sh`
    -   既存のインスタンスを終了
    -   Swift ビルド + パッケージング
    -   LaunchAgent の書き込み/ブートストラップ/起動
-   単一インスタンス: 同じバンドル ID を持つ別のインスタンスが実行中の場合、アプリは早期に終了します。

## セキュリティ強化に関する注意点

-   すべての特権操作に対して TeamID の一致を要求することを推奨します。
-   PeekabooBridge: `PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1` (DEBUG 専用) は、ローカル開発のために同一 UID の呼び出し元を許可する場合があります。
-   すべての通信はローカルのみに留まります。ネットワークソケットは公開されません。
-   TCC プロンプトは GUI アプリバンドルのみから発行されます。署名付きバンドル ID はリビルド後も安定していることを保証してください。
-   IPC 強化: ソケットモード `0600`、トークン、ピア UID チェック、HMAC チャレンジ/レスポンス、短い TTL。

[macOS 上の Gateway](./bundled-gateway.md)[スキル](./skills.md)