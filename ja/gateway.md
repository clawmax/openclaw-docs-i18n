

  Gateway

  
# Gateway ランブック

このページは、Gateway サービスの初回起動（デイ1）および継続運用（デイ2）に使用してください。

## 5分でのローカル起動

### ステップ 1: Gateway を起動する

```bash
openclaw gateway --port 18789
# debug/trace は stdio にもミラーリング
openclaw gateway --port 18789 --verbose
# 選択したポートのリスナーを強制終了してから起動
openclaw gateway --force
```

### ステップ 2: サービスヘルスを確認する

```bash
openclaw gateway status
openclaw status
openclaw logs --follow
```

健全な基準: `Runtime: running` および `RPC probe: ok`。

### ステップ 3: チャネルの準備状態を検証する

```bash
openclaw channels status --probe
```

 

> **ℹ️** Gateway 設定のリロードは、アクティブな設定ファイルパス（プロファイル/状態のデフォルトから解決、または `OPENCLAW_CONFIG_PATH` が設定されている場合はそれ）を監視します。デフォルトモードは `gateway.reload.mode="hybrid"` です。

## ランタイムモデル

-   ルーティング、コントロールプレーン、チャネル接続用の常時稼働プロセスが1つ。
-   単一の多重化ポートが以下を提供:
    -   WebSocket コントロール/RPC
    -   HTTP API (OpenAI互換、Responses、ツール呼び出し)
    -   コントロールUIとフック
-   デフォルトバインドモード: `loopback`。
-   デフォルトでは認証が必要 (`gateway.auth.token` / `gateway.auth.password`、または `OPENCLAW_GATEWAY_TOKEN` / `OPENCLAW_GATEWAY_PASSWORD`)。

### ポートとバインドの優先順位

| 設定 | 解決順序 |
| --- | --- |
| Gateway ポート | `--port` → `OPENCLAW_GATEWAY_PORT` → `gateway.port` → `18789` |
| バインドモード | CLI/オーバーライド → `gateway.bind` → `loopback` |

### ホットリロードモード

| `gateway.reload.mode` | 動作 |
| --- | --- |
| `off` | 設定リロードなし |
| `hot` | ホットセーフな変更のみ適用 |
| `restart` | リロードが必要な変更時に再起動 |
| `hybrid` (デフォルト) | 安全な場合はホット適用、必要な場合は再起動 |

## オペレーターコマンドセット

```bash
openclaw gateway status
openclaw gateway status --deep
openclaw gateway status --json
openclaw gateway install
openclaw gateway restart
openclaw gateway stop
openclaw secrets reload
openclaw logs --follow
openclaw doctor
```

## リモートアクセス

推奨: Tailscale/VPN。代替: SSH トンネル。

```bash
ssh -N -L 18789:127.0.0.1:18789 user@host
```

その後、クライアントをローカルの `ws://127.0.0.1:18789` に接続します。

> **⚠️** ゲートウェイ認証が設定されている場合、クライアントは SSH トンネル経由でも認証 (`token`/`password`) を送信する必要があります。

参照: [リモート Gateway](./gateway/remote.md), [認証](./gateway/authentication.md), [Tailscale](./gateway/tailscale.md)。

## 監視とサービスのライフサイクル

本番環境に近い信頼性のために、監視付き実行を使用してください。

].service\nopenclaw gateway status', lang: 'bash' }, { label: 'Linux (system service)', code: 'sudo systemctl daemon-reload\nsudo systemctl enable --now openclaw-gateway[-].service', lang: 'bash' }]} />

## 1ホスト上の複数ゲートウェイ

ほとんどのセットアップでは、**1つ**の Gateway を実行すべきです。厳密な分離/冗長性（例えばレスキュープロファイル）の場合のみ複数を使用してください。インスタンスごとのチェックリスト:

-   一意の `gateway.port`
-   一意の `OPENCLAW_CONFIG_PATH`
-   一意の `OPENCLAW_STATE_DIR`
-   一意の `agents.defaults.workspace`

例:

```
OPENCLAW_CONFIG_PATH=~/.openclaw/a.json OPENCLAW_STATE_DIR=~/.openclaw-a openclaw gateway --port 19001
OPENCLAW_CONFIG_PATH=~/.openclaw/b.json OPENCLAW_STATE_DIR=~/.openclaw-b openclaw gateway --port 19002
```

参照: [複数ゲートウェイ](./gateway/multiple-gateways.md)。

### 開発プロファイルのクイックパス

```bash
openclaw --dev setup
openclaw --dev gateway --allow-unconfigured
openclaw --dev status
```

デフォルトには、分離された状態/設定とベースゲートウェイポート `19001` が含まれます。

## プロトコルクイックリファレンス (オペレーター視点)

-   最初のクライアントフレームは `connect` でなければならない。
-   Gateway は `hello-ok` スナップショット (`presence`, `health`, `stateVersion`, `uptimeMs`, 制限/ポリシー) を返す。
-   リクエスト: `req(method, params)` → `res(ok/payload|error)`。
-   一般的なイベント: `connect.challenge`, `agent`, `chat`, `presence`, `tick`, `health`, `heartbeat`, `shutdown`。

エージェント実行は2段階:

1.  即時受理応答 (`status:"accepted"`)
2.  最終完了応答 (`status:"ok"|"error"`)、その間に `agent` イベントがストリーミングされる。

完全なプロトコルドキュメント参照: [Gateway プロトコル](./gateway/protocol.md)。

## 運用チェック

### 生存性

-   WS を開き、`connect` を送信。
-   スナップショット付きの `hello-ok` 応答を期待。

### 準備完了状態

```bash
openclaw gateway status
openclaw channels status --probe
openclaw health
```

### ギャップ回復

イベントは再生されません。シーケンスギャップ発生時は、続行前に状態 (`health`, `system-presence`) を更新してください。

## 一般的な障害シグネチャ

| シグネチャ | 可能性の高い問題 |
| --- | --- |
| `refusing to bind gateway ... without auth` | トークン/パスワードなしでの非ループバックバインド |
| `another gateway instance is already listening` / `EADDRINUSE` | ポート競合 |
| `Gateway start blocked: set gateway.mode=local` | 設定がリモートモードに設定されている |
| `unauthorized` (接続中) | クライアントとゲートウェイ間の認証不一致 |

完全な診断ラダーについては、[Gateway トラブルシューティング](./gateway/troubleshooting.md)を使用してください。

## 安全性の保証

-   Gateway プロトコルクライアントは、Gateway が利用できない場合に高速に失敗する（暗黙的な直接チャネルフォールバックなし）。
-   無効な/非接続の最初のフレームは拒否され、クローズされる。
-   グレースフルシャットダウンは、ソケットクローズ前に `shutdown` イベントを発行する。

* * *

関連項目:

-   [トラブルシューティング](./gateway/troubleshooting.md)
-   [バックグラウンドプロセス](./gateway/background-process.md)
-   [設定](./gateway/configuration.md)
-   [ヘルス](./gateway/health.md)
-   [Doctor](./gateway/doctor.md)
-   [認証](./gateway/authentication.md)

[設定](./gateway/configuration.md)