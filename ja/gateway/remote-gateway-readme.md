

  リモートアクセス

  
# リモートゲートウェイ設定

OpenClaw.appはリモートゲートウェイへの接続にSSHトンネリングを使用します。このガイドではその設定方法を説明します。

## 概要

## クイックセットアップ

### ステップ 1: SSH設定を追加

`~/.ssh/config`を編集し、以下を追加します:

```
Host remote-gateway
    HostName <REMOTE_IP>          # 例: 172.27.187.184
    User <REMOTE_USER>            # 例: jefferson
    LocalForward 18789 127.0.0.1:18789
    IdentityFile ~/.ssh/id_rsa
```

`<REMOTE_IP>`と`<REMOTE_USER>`を実際の値に置き換えてください。

### ステップ 2: SSHキーをコピー

公開鍵をリモートマシンにコピーします（パスワードを一度入力）:

```bash
ssh-copy-id -i ~/.ssh/id_rsa <REMOTE_USER>@<REMOTE_IP>
```

### ステップ 3: ゲートウェイトークンを設定

```bash
launchctl setenv OPENCLAW_GATEWAY_TOKEN "<your-token>"
```

### ステップ 4: SSHトンネルを開始

```bash
ssh -N remote-gateway &
```

### ステップ 5: OpenClaw.appを再起動

```bash
# OpenClaw.appを終了（⌘Q）し、再度開きます:
open /path/to/OpenClaw.app
```

これでアプリはSSHトンネルを介してリモートゲートウェイに接続します。

* * *

## ログイン時にトンネルを自動起動

ログイン時にSSHトンネルを自動的に起動するには、Launch Agentを作成します。

### PLISTファイルを作成

これを`~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist`として保存します:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ai.openclaw.ssh-tunnel</string>
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/ssh</string>
        <string>-N</string>
        <string>remote-gateway</string>
    </array>
    <key>KeepAlive</key>
    <true/>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

### Launch Agentをロード

```bash
launchctl bootstrap gui/$UID ~/Library/LaunchAgents/ai.openclaw.ssh-tunnel.plist
```

これでトンネルは以下のようになります:

-   ログイン時に自動的に起動
-   クラッシュした場合に再起動
-   バックグラウンドで実行を継続

レガシーノート: 残っている`com.openclaw.ssh-tunnel` LaunchAgentがあれば削除してください。

* * *

## トラブルシューティング

**トンネルが実行中か確認:**

```bash
ps aux | grep "ssh -N remote-gateway" | grep -v grep
lsof -i :18789
```

**トンネルを再起動:**

```bash
launchctl kickstart -k gui/$UID/ai.openclaw.ssh-tunnel
```

**トンネルを停止:**

```bash
launchctl bootout gui/$UID/ai.openclaw.ssh-tunnel
```

* * *

## 仕組み

| コンポーネント | 機能 |
| --- | --- |
| `LocalForward 18789 127.0.0.1:18789` | ローカルポート18789をリモートポート18789に転送 |
| `ssh -N` | リモートコマンドを実行しないSSH（ポートフォワーディングのみ） |
| `KeepAlive` | トンネルがクラッシュした場合に自動的に再起動 |
| `RunAtLoad` | エージェントロード時にトンネルを起動 |

OpenClaw.appはクライアントマシンの`ws://127.0.0.1:18789`に接続します。SSHトンネルはその接続を、ゲートウェイが実行されているリモートマシンのポート18789に転送します。

[リモートアクセス](./remote.md)[Tailscale](./tailscale.md)