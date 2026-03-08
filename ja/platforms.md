

  プラットフォーム概要

  
# プラットフォーム

OpenClawのコアはTypeScriptで書かれています。**推奨ランタイムはNodeです**。Gateway（WhatsApp/Telegramのバグのため）にはBunは推奨されません。コンパニオンアプリはmacOS（メニューバーアプリ）とモバイルノード（iOS/Android）向けに存在します。WindowsとLinuxのコンパニオンアプリは計画中ですが、Gatewayは現在完全にサポートされています。Windows向けのネイティブコンパニオンアプリも計画中です。現時点ではWSL2経由でのGatewayの利用が推奨されます。

## OSを選択

-   macOS: [macOS](./platforms/macos.md)
-   iOS: [iOS](./platforms/ios.md)
-   Android: [Android](./platforms/android.md)
-   Windows: [Windows](./platforms/windows.md)
-   Linux: [Linux](./platforms/linux.md)

## VPS & ホスティング

-   VPSハブ: [VPSホスティング](./vps.md)
-   Fly.io: [Fly.io](./install/fly.md)
-   Hetzner (Docker): [Hetzner](./install/hetzner.md)
-   GCP (Compute Engine): [GCP](./install/gcp.md)
-   exe.dev (VM + HTTPSプロキシ): [exe.dev](./install/exe-dev.md)

## よく使われるリンク

-   インストールガイド: [はじめに](./start/getting-started.md)
-   Gateway運用マニュアル: [Gateway](./gateway.md)
-   Gateway設定: [設定](./gateway/configuration.md)
-   サービスステータス: `openclaw gateway status`

## Gatewayサービスインストール (CLI)

以下のいずれかを使用してください（すべてサポートされています）:

-   ウィザード（推奨）: `openclaw onboard --install-daemon`
-   直接インストール: `openclaw gateway install`
-   設定フロー: `openclaw configure` → **Gatewayサービス**を選択
-   修復/移行: `openclaw doctor` (サービスのインストールまたは修復を提案します)

サービスのターゲットはOSによって異なります:

-   macOS: LaunchAgent (`ai.openclaw.gateway` または `ai.openclaw.`; レガシー `com.openclaw.*`)
-   Linux/WSL2: systemdユーザーサービス (`openclaw-gateway[-].service`)

[macOSアプリ](./platforms/macos.md)