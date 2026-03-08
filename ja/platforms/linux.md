

  プラットフォーム概要

  
# Linux アプリ

Gateway は Linux で完全にサポートされています。**Node が推奨ランタイムです**。Gateway での Bun は推奨されません（WhatsApp/Telegram のバグ）。ネイティブ Linux コンパニオンアプリは計画中です。構築に協力したい場合は、コントリビューションを歓迎します。

## 初心者向けクイックパス (VPS)

1.  Node 22+ をインストール
2.  `npm i -g openclaw@latest`
3.  `openclaw onboard --install-daemon`
4.  ラップトップから: `ssh -N -L 18789:127.0.0.1:18789 <ユーザー>@<ホスト>`
5.  `http://127.0.0.1:18789/` を開き、トークンを貼り付け

ステップバイステップ VPS ガイド: [exe.dev](../install/exe-dev.md)

## インストール

-   [はじめに](../start/getting-started.md)
-   [インストール & 更新](../install/updating.md)
-   オプションフロー: [Bun (実験的)](../install/bun.md), [Nix](../install/nix.md), [Docker](../install/docker.md)

## ゲートウェイ

-   [ゲートウェイ ランブック](../gateway.md)
-   [設定](../gateway/configuration.md)

## ゲートウェイサービスインストール (CLI)

以下のいずれかを使用:

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

プロンプトが表示されたら **Gateway service** を選択。修復/移行:

```bash
openclaw doctor
```

## システム制御 (systemd ユニット)

OpenClaw はデフォルトで systemd **ユーザー** サービスをインストールします。共有サーバーや常時稼働サーバーには **システム** サービスを使用してください。完全なユニット例とガイダンスは [ゲートウェイ ランブック](../gateway.md) にあります。最小セットアップ: `~/.config/systemd/user/openclaw-gateway[-<プロファイル>].service` を作成:

```ini
[Unit]
Description=OpenClaw Gateway (profile: <profile>, v<version>)
After=network-online.target
Wants=network-online.target

[Service]
ExecStart=/usr/local/bin/openclaw gateway --port 18789
Restart=always
RestartSec=5

[Install]
WantedBy=default.target
```

有効化:

```bash
systemctl --user enable --now openclaw-gateway[-<profile>].service
```

[macOS アプリ](./macos.md)[Windows (WSL2)](./windows.md)