title: "メディアとデバイス向け OpenClaw ノードトラブルシューティングガイド"
description: "ノードの可視性の問題、権限エラー、バックグラウンドの問題を修正します。iOS、Android、macOS ノードの診断コマンドの使用方法と一般的なエラーコードの解決方法を学びます。"
keywords: ["ノードトラブルシューティング", "openclawノード", "権限エラー", "ノードバックグラウンド利用不可", "実行承認", "デバイスペアリング", "ノードエラーコード", "カメラ権限"]
---

  メディアとデバイス

  
# ノードトラブルシューティング

ノードがステータスでは表示されるが、ノードツールが失敗する場合にこのページを使用してください。

## コマンドラダー

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

次に、ノード固有のチェックを実行します:

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

正常なシグナル:

-   ノードが接続され、ロール `node` としてペアリングされている。
-   `nodes describe` に呼び出している機能が含まれている。
-   実行承認が期待されるモード/許可リストを表示する。

## フォアグラウンド要件

`canvas.*`、`camera.*`、`screen.*` は、iOS/Android ノードではフォアグラウンドのみで動作します。クイックチェックと修正:

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

`NODE_BACKGROUND_UNAVAILABLE` が表示された場合は、ノードアプリをフォアグラウンドに戻して再試行してください。

## 権限マトリックス

| 機能 | iOS | Android | macOS ノードアプリ | 典型的な失敗コード |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | カメラ (+ クリップ音声用マイク) | カメラ (+ クリップ音声用マイク) | カメラ (+ クリップ音声用マイク) | `*_PERMISSION_REQUIRED` |
| `screen.record` | 画面収録 (+ マイクはオプション) | 画面キャプチャプロンプト (+ マイクはオプション) | 画面収録 | `*_PERMISSION_REQUIRED` |
| `location.get` | 使用中または常時 (モードによる) | モードに基づくフォアグラウンド/バックグラウンド位置情報 | 位置情報権限 | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | n/a (ノードホストパス) | n/a (ノードホストパス) | 実行承認が必要 | `SYSTEM_RUN_DENIED` |

## ペアリングと承認の違い

これらは異なるゲートです:

1.  **デバイスペアリング**: このノードはゲートウェイに接続できますか？
2.  **実行承認**: このノードは特定のシェルコマンドを実行できますか？

クイックチェック:

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

ペアリングが欠落している場合は、まずノードデバイスを承認してください。ペアリングは正常だが `system.run` が失敗する場合は、実行承認/許可リストを修正してください。

## 一般的なノードエラーコード

-   `NODE_BACKGROUND_UNAVAILABLE` → アプリがバックグラウンド化されています。フォアグラウンドに戻してください。
-   `CAMERA_DISABLED` → ノード設定でカメラトグルが無効になっています。
-   `*_PERMISSION_REQUIRED` → OS権限が不足している/拒否されています。
-   `LOCATION_DISABLED` → 位置情報モードがオフです。
-   `LOCATION_PERMISSION_REQUIRED` → 要求された位置情報モードが許可されていません。
-   `LOCATION_BACKGROUND_UNAVAILABLE` → アプリがバックグラウンド化されていますが、「使用中のみ」の権限しかありません。
-   `SYSTEM_RUN_DENIED: approval required` → 実行リクエストには明示的な承認が必要です。
-   `SYSTEM_RUN_DENIED: allowlist miss` → コマンドが許可リストモードによってブロックされています。Windows ノードホストでは、`cmd.exe /c ...` のようなシェルラッパー形式は、承認フローを経て承認されない限り、許可リストモードでは許可リストミスとして扱われます。

## 高速回復ループ

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

それでも解決しない場合:

-   デバイスペアリングを再承認する。
-   ノードアプリを再起動する（フォアグラウンドで）。
-   OS権限を再付与する。
-   実行承認ポリシーを再作成/調整する。

関連項目:

-   [/nodes/index](./index.md)
-   [/nodes/camera](./camera.md)
-   [/nodes/location-command](./location-command.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)
-   [/gateway/pairing](../gateway/pairing.md)

[ノード](../nodes.md)[メディア理解](./media-understanding.md)