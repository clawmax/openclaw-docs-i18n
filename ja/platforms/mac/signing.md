

  macOSコンパニオンアプリ

  
# macOS署名

このアプリは通常、[`scripts/package-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/package-mac-app.sh) からビルドされます。このスクリプトは現在、以下のことを行います：

-   安定したデバッグ用バンドル識別子を設定: `ai.openclaw.mac.debug`
-   そのバンドルIDでInfo.plistを書き込みます（`BUNDLE_ID=...`で上書き可能）
-   [`scripts/codesign-mac-app.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/codesign-mac-app.sh) を呼び出し、メインバイナリとアプリバンドルに署名します。これにより、macOSは各リビルドを同じ署名済みバンドルとして扱い、TCC権限（通知、アクセシビリティ、画面録画、マイク、音声認識）を保持します。安定した権限のためには、実際の署名IDを使用してください。アドホック署名はオプトインであり、脆弱です（詳細は [macOS権限](./permissions.md) を参照）。
-   デフォルトで `CODESIGN_TIMESTAMP=auto` を使用します。これはDeveloper ID署名に対して信頼できるタイムスタンプを有効にします。タイムスタンプをスキップするには `CODESIGN_TIMESTAMP=off` を設定します（オフラインのデバッグビルド用）。
-   Info.plistにビルドメタデータを注入します: `OpenClawBuildTimestamp` (UTC) と `OpenClawGitCommit` (短縮ハッシュ)。これにより「このアプリについて」ペインで、ビルド、git、デバッグ/リリースチャネルを表示できます。
-   **パッケージングにはNode 22+が必要です**: スクリプトはTSビルドとControl UIビルドを実行します。
-   環境変数から `SIGN_IDENTITY` を読み取ります。常にあなたの証明書で署名するには、シェルのrcファイルに `export SIGN_IDENTITY="Apple Development: Your Name (TEAMID)"`（またはあなたのDeveloper ID Application証明書）を追加してください。アドホック署名には、`ALLOW_ADHOC_SIGNING=1` または `SIGN_IDENTITY="-"` による明示的なオプトインが必要です（権限テストには非推奨）。
-   署名後にTeam ID監査を実行し、アプリバンドル内のMach-Oが異なるTeam IDで署名されている場合に失敗します。チェックをバイパスするには `SKIP_TEAM_ID_CHECK=1` を設定します。

## 使用方法

```bash
# リポジトリのルートから
scripts/package-mac-app.sh               # 署名IDを自動選択。見つからない場合はエラー
SIGN_IDENTITY="Developer ID Application: Your Name" scripts/package-mac-app.sh   # 実際の証明書
ALLOW_ADHOC_SIGNING=1 scripts/package-mac-app.sh    # アドホック（権限は保持されません）
SIGN_IDENTITY="-" scripts/package-mac-app.sh        # 明示的なアドホック（同じ注意点）
DISABLE_LIBRARY_VALIDATION=1 scripts/package-mac-app.sh   # 開発専用：SparkleのTeam ID不一致回避策
```

### アドホック署名に関する注意

`SIGN_IDENTITY="-"`（アドホック）で署名する場合、スクリプトは自動的に **ハードニングランタイム** (`--options runtime`) を無効にします。これは、同じTeam IDを共有しない埋め込みフレームワーク（Sparkleなど）をアプリがロードしようとした際のクラッシュを防ぐために必要です。アドホック署名はTCC権限の永続性も損ないます。回復手順については [macOS権限](./permissions.md) を参照してください。

## 「このアプリについて」のためのビルドメタデータ

`package-mac-app.sh` はバンドルに以下の情報をスタンプします：

-   `OpenClawBuildTimestamp`: パッケージング時のISO8601形式のUTC
-   `OpenClawGitCommit`: 短縮gitハッシュ（利用不可の場合は `unknown`）

「このアプリについて」タブはこれらのキーを読み取り、バージョン、ビルド日時、gitコミット、およびデバッグビルドかどうか（`#if DEBUG` 経由）を表示します。コード変更後は、これらの値を更新するためにパッケージャーを実行してください。

## 理由

TCC権限は、バンドル識別子*と*コード署名に紐づいています。UUIDが変わる未署名のデバッグビルドでは、macOSがリビルドのたびに権限付与を忘れてしまう問題がありました。バイナリに署名し（デフォルトはアドホック）、固定のバンドルID/パス（`dist/OpenClaw.app`）を維持することで、VibeTunnelのアプローチと同様に、ビルド間で権限付与を保持できるようになります。

[リモートコントロール](./remote.md)[macOSリリース](./release.md)