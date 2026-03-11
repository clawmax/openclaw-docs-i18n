

  macOS コンパニオンアプリ

  
# Peekaboo Bridge

OpenClawは、ローカルで動作し、権限を認識するUI自動化ブローカーとして**PeekabooBridge**をホストできます。これにより、`peekaboo` CLIがmacOSアプリのTCC権限を再利用しながらUI自動化を駆動できるようになります。

## これは何か（そして何ではないか）

-   **ホスト**: OpenClaw.appはPeekabooBridgeホストとして機能できます。
-   **クライアント**: `peekaboo` CLIを使用します（別途`openclaw ui ...`というインターフェースはありません）。
-   **UI**: ビジュアルオーバーレイはPeekaboo.appに残ります。OpenClawはシンなブローカーホストです。

## ブリッジを有効にする

macOSアプリ内で:

-   設定 → **Peekaboo Bridgeを有効にする**

有効にすると、OpenClawはローカルのUNIXソケットサーバーを起動します。無効にすると、ホストは停止し、`peekaboo`は他の利用可能なホストにフォールバックします。

## クライアントの検出順序

Peekabooクライアントは通常、以下の順序でホストを試行します:

1.  Peekaboo.app (完全なUX)
2.  Claude.app (インストールされている場合)
3.  OpenClaw.app (シンなブローカー)

`peekaboo bridge status --verbose`を使用して、どのホストがアクティブで、どのソケットパスが使用されているかを確認できます。以下のようにして上書きできます:

```bash
export PEEKABOO_BRIDGE_SOCKET=/path/to/bridge.sock
```

## セキュリティと権限

-   ブリッジは**呼び出し元のコード署名**を検証します。TeamIDの許可リストが適用されます（PeekabooホストのTeamID + OpenClawアプリのTeamID）。
-   リクエストは約10秒後にタイムアウトします。
-   必要な権限が不足している場合、ブリッジはシステム設定を起動するのではなく、明確なエラーメッセージを返します。

## スナップショットの動作（自動化）

スナップショットはメモリに保存され、短時間後に自動的に期限切れになります。より長い保持期間が必要な場合は、クライアントから再キャプチャしてください。

## トラブルシューティング

-   `peekaboo`が「bridge client is not authorized」と報告する場合、クライアントが適切に署名されていることを確認するか、**デバッグ**モードでのみ`PEEKABOO_ALLOW_UNSIGNED_SOCKET_CLIENTS=1`を設定してホストを実行してください。
-   ホストが見つからない場合は、ホストアプリ（Peekaboo.appまたはOpenClaw.app）のいずれかを開き、権限が付与されていることを確認してください。

[スキル](./skills.md)

---