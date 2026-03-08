

  プラットフォーム概要

  
# Windows (WSL2)

Windows での OpenClaw は **WSL2経由**（Ubuntu推奨）での利用を推奨します。CLIとゲートウェイはLinux内部で動作し、これによりランタイムが一貫し、ツール（Node/Bun/pnpm、Linuxバイナリ、スキル）との互換性が大幅に向上します。ネイティブWindowsはより複雑になる可能性があります。WSL2は完全なLinux環境を提供します — インストールは一つのコマンド: `wsl --install` です。ネイティブWindows用コンパニオンアプリは計画中です。

## インストール (WSL2)

-   [はじめに](../start/getting-started.md) (WSL内部で使用)
-   [インストールと更新](../install/updating.md)
-   WSL2公式ガイド (Microsoft): [https://learn.microsoft.com/windows/wsl/install](https://learn.microsoft.com/windows/wsl/install)

## ゲートウェイ

-   [ゲートウェイ運用手順書](../gateway.md)
-   [設定](../gateway/configuration.md)

## ゲートウェイサービスのインストール (CLI)

WSL2内部で:

```bash
openclaw onboard --install-daemon
```

または:

```bash
openclaw gateway install
```

または:

```bash
openclaw configure
```

プロンプトが表示されたら **Gateway service** を選択してください。修復/移行:

```bash
openclaw doctor
```

## Windowsログイン前のゲートウェイ自動起動

ヘッドレスセットアップでは、誰もWindowsにログインしなくても完全な起動チェーンが実行されるようにします。

### 1) ログインなしでユーザーサービスを実行し続ける

WSL内部で:

```bash
sudo loginctl enable-linger "$(whoami)"
```

### 2) OpenClawゲートウェイユーザーサービスをインストール

WSL内部で:

```bash
openclaw gateway install
```

### 3) Windows起動時にWSLを自動的に開始

管理者としてPowerShellで:

```bash
schtasks /create /tn "WSL Boot" /tr "wsl.exe -d Ubuntu --exec /bin/true" /sc onstart /ru SYSTEM
```

`Ubuntu` を以下のコマンドで確認したディストリビューション名に置き換えてください:

```bash
wsl --list --verbose
```

### 起動チェーンの確認

再起動後（Windowsサインイン前）、WSLから確認:

```bash
systemctl --user is-enabled openclaw-gateway
systemctl --user status openclaw-gateway --no-pager
```

## 上級: WSLサービスをLANに公開 (portproxy)

WSLは独自の仮想ネットワークを持っています。別のマシンが **WSL内部** で動作するサービス（SSH、ローカルTTSサーバー、またはゲートウェイ）に到達する必要がある場合、Windowsのポートを現在のWSL IPに転送する必要があります。WSL IPは再起動後に変更されるため、転送ルールを更新する必要がある場合があります。例（管理者としての **PowerShell**）:

```powershell
$Distro = "Ubuntu-24.04"
$ListenPort = 2222
$TargetPort = 22

$WslIp = (wsl -d $Distro -- hostname -I).Trim().Split(" ")[0]
if (-not $WslIp) { throw "WSL IP not found." }

netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=$ListenPort `
  connectaddress=$WslIp connectport=$TargetPort
```

Windowsファイアウォールでポートを許可（一度だけ）:

```
New-NetFirewallRule -DisplayName "WSL SSH $ListenPort" -Direction Inbound `
  -Protocol TCP -LocalPort $ListenPort -Action Allow
```

WSL再起動後にportproxyを更新:

```
netsh interface portproxy delete v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 | Out-Null
netsh interface portproxy add v4tov4 listenport=$ListenPort listenaddress=0.0.0.0 `
  connectaddress=$WslIp connectport=$TargetPort | Out-Null
```

注意点:

-   別のマシンからのSSHは **WindowsホストIP** をターゲットにします（例: `ssh user@windows-host -p 2222`）。
-   リモートノードは **到達可能な** ゲートウェイURLを指す必要があります（`127.0.0.1`ではない）。確認には `openclaw status --all` を使用してください。
-   LANアクセスの場合は `listenaddress=0.0.0.0` を使用。`127.0.0.1` はローカルのみに制限します。
-   これを自動化したい場合は、ログイン時に更新ステップを実行するスケジュールタスクを登録してください。

## ステップバイステップ WSL2 インストール

### 1) WSL2 + Ubuntu のインストール

管理者としてPowerShellを開く:

```bash
wsl --install
# または明示的にディストリビューションを選択:
wsl --list --online
wsl --install -d Ubuntu-24.04
```

Windowsが要求した場合は再起動します。

### 2) systemd を有効化 (ゲートウェイインストールに必須)

WSLターミナルで:

```bash
sudo tee /etc/wsl.conf >/dev/null <<'EOF'
[boot]
systemd=true
EOF
```

次にPowerShellから:

```bash
wsl --shutdown
```

Ubuntuを再度開き、確認:

```bash
systemctl --user status
```

### 3) OpenClaw のインストール (WSL内部)

WSL内部でLinuxの「はじめに」フローに従います:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build # 初回実行時にUIの依存関係を自動インストール
pnpm build
openclaw onboard
```

完全なガイド: [はじめに](../start/getting-started.md)

## Windowsコンパニオンアプリ

現在、Windowsコンパニオンアプリはありません。実現に向けた貢献を歓迎します。

[Linuxアプリ](./linux.md)[Androidアプリ](./android.md)