

  macOS コンパニオンアプリ

  
# macOS リリース

このアプリは現在、Sparkle 自動更新を同梱しています。リリースビルドは Developer ID で署名し、zip 圧縮し、署名付きの appcast エントリと共に公開する必要があります。

## 前提条件

-   Developer ID Application 証明書がインストール済み（例: `Developer ID Application:  ()`）。
-   Sparkle 秘密鍵のパスが環境変数 `SPARKLE_PRIVATE_KEY_FILE` に設定済み（Sparkle ed25519 秘密鍵へのパス。公開鍵は Info.plist に組み込まれています）。見つからない場合は `~/.profile` を確認してください。
-   Gatekeeper 対応の DMG/zip 配布を行う場合、`xcrun notarytool` 用の公証資格情報（キーチェーンプロファイルまたは API キー）。
    -   シェルプロファイルの App Store Connect API キー環境変数から作成された `openclaw-notary` という名前のキーチェーンプロファイルを使用しています:
        -   `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
        -   `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
        -   `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
-   `pnpm` 依存関係がインストール済み (`pnpm install --config.node-linker=hoisted`)。
-   Sparkle ツールは SwiftPM 経由で `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` に自動的に取得されます (`sign_update`, `generate_appcast` など)。

## ビルド & パッケージ化

注意点:

-   `APP_BUILD` は `CFBundleVersion`/`sparkle:version` に対応します。数値で単調増加するようにしてください（`-beta` などは付けない）。そうしないと Sparkle が同等とみなして比較します。
-   `APP_BUILD` が省略された場合、`scripts/package-mac-app.sh` は `APP_VERSION` から Sparkle 対応のデフォルト値を導出します (`YYYYMMDDNN`: 安定版は `90` をデフォルトとし、プレリリース版はサフィックスから導出されたレーンを使用します)。そして、その値と git コミット数の大きい方を使用します。
-   リリースエンジニアリングで特定の単調増加値が必要な場合は、明示的に `APP_BUILD` を上書きできます。
-   デフォルトは現在のアーキテクチャ (`$(uname -m)`) です。リリース/ユニバーサルビルドの場合は、`BUILD_ARCHS="arm64 x86_64"` (または `BUILD_ARCHS=all`) を設定してください。
-   リリース成果物 (zip + DMG + 公証) には `scripts/package-mac-dist.sh` を使用してください。ローカル/開発用のパッケージングには `scripts/package-mac-app.sh` を使用してください。

```bash
# リポジトリのルートから実行。Sparkle フィードが有効になるようリリース ID を設定。
# APP_BUILD は Sparkle の比較のために数値で単調増加する必要があります。
# 省略時は APP_VERSION から自動導出されます。
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# 配布用に zip 化 (Sparkle デルタ更新サポートのためリソースフォークを含む)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.3.7.zip

# オプション: 人間向けにスタイル付き DMG も作成 (/Applications にドラッグ)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.3.7.dmg

# 推奨: zip + DMG のビルド + 公証/ステープル
# 最初に、キーチェーンプロファイルを一度作成:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# オプション: リリースに dSYM を同梱
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.3.7.dSYM.zip
```

## Appcast エントリ

Sparkle がフォーマットされた HTML リリースノートをレンダリングするよう、リリースノートジェネレーターを使用してください:

```
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.3.7.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

`CHANGELOG.md` から HTML リリースノートを生成し ([`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh) 経由)、それを appcast エントリに埋め込みます。公開時には、リリースアセット (zip + dSYM) と共に更新された `appcast.xml` をコミットしてください。

## 公開 & 検証

-   `OpenClaw-2026.3.7.zip` (および `OpenClaw-2026.3.7.dSYM.zip`) を、タグ `v2026.3.7` の GitHub リリースにアップロードします。
-   生の appcast URL が組み込みフィードと一致していることを確認します: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`。
-   健全性チェック:
    -   `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` が 200 を返す。
    -   アセットアップロード後、`curl -I ` が 200 を返す。
    -   以前の公開ビルドをインストールした環境で、About タブから「アップデートを確認…」を実行し、Sparkle が新しいビルドをクリーンにインストールすることを確認する。

完了の定義: 署名済みアプリと appcast が公開され、古いインストール済みバージョンからの更新フローが機能し、リリースアセットが GitHub リリースに添付されていること。

[macOS 署名](./signing.md)[macOS 上のゲートウェイ](./bundled-gateway.md)