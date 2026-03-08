title: "OpenClaw Android アプリセットアップガイドとプラットフォーム概要"
description: "OpenClaw Android コンパニオンノードのセットアップ方法、Gateway への接続方法、チャット、キャンバス、カメラ、音声などの機能の使用方法を学びます。"
keywords: ["openclaw android", "android ノード セットアップ", "ゲートウェイ ペアリング", "android コンパニオン アプリ", "openclaw ゲートウェイ", "android キャンバス", "android 音声制御", "tailscale android"]
---

  プラットフォーム概要

  
# Android アプリ

## サポートスナップショット

-   役割: コンパニオンノードアプリ (Android は Gateway をホストしません)。
-   Gateway 必須: はい (macOS、Linux、または Windows via WSL2 で実行します)。
-   インストール: [はじめに](../start/getting-started.md) + [ペアリング](../channels/pairing.md)。
-   Gateway: [運用手順書](../gateway.md) + [設定](../gateway/configuration.md)。
    -   プロトコル: [Gateway プロトコル](../gateway/protocol.md) (ノード + コントロールプレーン)。

## システム制御

システム制御 (launchd/systemd) は Gateway ホスト上にあります。[Gateway](../gateway.md) を参照してください。

## 接続運用手順書

Android ノードアプリ ⇄ (mDNS/NSD + WebSocket) ⇄ **Gateway** Android は Gateway WebSocket (デフォルト `ws://:18789`) に直接接続し、デバイスペアリング (`role: node`) を使用します。

### 前提条件

-   「マスター」マシンで Gateway を実行できること。
-   Android デバイス/エミュレーターが gateway WebSocket に到達できること:
    -   同じ LAN で mDNS/NSD を使用、**または**
    -   同じ Tailscale tailnet で Wide-Area Bonjour / ユニキャスト DNS-SD を使用 (下記参照)、**または**
    -   手動 gateway ホスト/ポート (フォールバック)
-   gateway マシン上 (または SSH 経由で) CLI (`openclaw`) を実行できること。

### 1) Gateway を起動する

```bash
openclaw gateway --port 18789 --verbose
```

ログで以下のような表示を確認してください:

-   `listening on ws://0.0.0.0:18789`

tailnet 専用セットアップ (Vienna ⇄ London に推奨) の場合、gateway を tailnet IP にバインドします:

-   gateway ホスト上の `~/.openclaw/openclaw.json` で `gateway.bind: "tailnet"` を設定します。
-   Gateway / macOS メニューバーアプリを再起動します。

### 2) ディスカバリーを確認する (オプション)

gateway マシンから:

```bash
dns-sd -B _openclaw-gw._tcp local.
```

詳細なデバッグメモ: [Bonjour](../gateway/bonjour.md)。

#### Tailnet (Vienna ⇄ London) ユニキャスト DNS-SD によるディスカバリー

Android NSD/mDNS ディスカバリーはネットワークを越えません。Android ノードと gateway が異なるネットワーク上にあるが Tailscale 経由で接続されている場合、代わりに Wide-Area Bonjour / ユニキャスト DNS-SD を使用します:

1.  gateway ホスト上に DNS-SD ゾーン (例 `openclaw.internal.`) を設定し、`_openclaw-gw._tcp` レコードを公開します。
2.  選択したドメインに対して、その DNS サーバーを指す Tailscale スプリット DNS を設定します。

詳細と CoreDNS 設定例: [Bonjour](../gateway/bonjour.md)。

### 3) Android から接続する

Android アプリ内で:

-   アプリは **フォアグラウンドサービス** (永続的な通知) を介して gateway 接続を維持します。
-   **接続** タブを開きます。
-   **セットアップコード** または **手動** モードを使用します。
-   ディスカバリーがブロックされている場合は、**詳細設定** で手動ホスト/ポート (および必要に応じて TLS/トークン/パスワード) を使用します。

初回ペアリング成功後、Android は起動時に自動再接続します:

-   手動エンドポイント (有効な場合)、それ以外は
-   最後に発見された gateway (ベストエフォート)。

### 4) ペアリングを承認する (CLI)

gateway マシン上で:

```bash
openclaw devices list
openclaw devices approve <requestId>
openclaw devices reject <requestId>
```

ペアリングの詳細: [ペアリング](../channels/pairing.md)。

### 5) ノードが接続されていることを確認する

-   ノードステータス経由:
    
    コピー
    
    ```bash
    openclaw nodes status
    ```
    
-   Gateway 経由:
    
    コピー
    
    ```bash
    openclaw gateway call node.list --params "{}"
    ```
    

### 6) チャット + 履歴

Android のチャットタブはセッション選択 (デフォルト `main`、およびその他の既存セッション) をサポートします:

-   履歴: `chat.history`
-   送信: `chat.send`
-   プッシュ更新 (ベストエフォート): `chat.subscribe` → `event:"chat"`

### 7) キャンバス + 画面 + カメラ

#### Gateway キャンバスホスト (Web コンテンツに推奨)

ノードにエージェントがディスク上で編集できる実際の HTML/CSS/JS を表示させたい場合は、ノードを Gateway キャンバスホストに向けます。注: ノードは Gateway HTTP サーバー (`gateway.port` と同じポート、デフォルト `18789`) からキャンバスを読み込みます。

1.  gateway ホスト上に `~/.openclaw/workspace/canvas/index.html` を作成します。
2.  ノードをそこにナビゲートします (LAN):

```bash
openclaw nodes invoke --node "<Android Node>" --command canvas.navigate --params '{"url":"http://<gateway-hostname>.local:18789/__openclaw__/canvas/"}'
```

Tailnet (オプション): 両方のデバイスが Tailscale 上にある場合、`.local` の代わりに MagicDNS 名または tailnet IP を使用します (例: `http://<gateway-magicdns>:18789/__openclaw__/canvas/`)。このサーバーは HTML にライブリロードクライアントを注入し、ファイル変更時にリロードします。A2UI ホストは `http://<gateway-host>:18789/__openclaw__/a2ui/` にあります。キャンバスコマンド (フォアグラウンドのみ):

-   `canvas.eval`, `canvas.snapshot`, `canvas.navigate` (`{"url":""}` または `{"url":"/"}` を使用してデフォルトのスキャフォールドに戻ります)。`canvas.snapshot` は `{ format, base64 }` を返します (デフォルト `format="jpeg"`)。
-   A2UI: `canvas.a2ui.push`, `canvas.a2ui.reset` (`canvas.a2ui.pushJSONL` はレガシーエイリアス)

カメラコマンド (フォアグラウンドのみ; パーミッション制限あり):

-   `camera.snap` (jpg)
-   `camera.clip` (mp4)

パラメータと CLI ヘルパーについては [カメラノード](../nodes/camera.md) を参照してください。画面コマンド:

-   `screen.record` (mp4; フォアグラウンドのみ)

### 8) 音声 + 拡張 Android コマンドサーフェス

-   音声: Android は音声タブで単一マイクのオン/オフフローを使用し、文字起こしキャプチャと TTS 再生 (設定時は ElevenLabs、システム TTS フォールバック) を行います。
-   音声ウェイク/トークモード切り替えは現在、Android UX/ランタイムから削除されています。
-   追加の Android コマンドファミリー (利用可否はデバイス + パーミッションに依存):
    -   `device.status`, `device.info`, `device.permissions`, `device.health`
    -   `notifications.list`, `notifications.actions`
    -   `photos.latest`
    -   `contacts.search`, `contacts.add`
    -   `calendar.events`, `calendar.add`
    -   `motion.activity`, `motion.pedometer`
    -   `app.update`

[Windows (WSL2)](./windows.md)[iOS アプリ](./ios.md)