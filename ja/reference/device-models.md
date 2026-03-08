title: "Appleデバイス識別子のためのOpenClawデバイスモデルデータベース"
description: "OpenClawがAppleデバイス識別子を人間が読みやすい名前にマッピングする方法について学びます。iOSおよびmacOSモデルデータベースのデータソースと更新手順を確認してください。"
keywords: ["appleデバイス識別子", "デバイスモデルデータベース", "iosモデル識別子", "macosモデル識別子", "openclawデバイスマッピング", "rpcデバイスモデル", "apiデバイス名", "appleモデル識別子マッピング"]
---

  RPCとAPI

  
# デバイスモデルデータベース

macOSコンパニオンアプリは、Appleモデル識別子（例: `iPad16,6`, `Mac16,6`）を人間が読みやすい名前にマッピングすることで、**インスタンス** UIにわかりやすいAppleデバイスモデル名を表示します。このマッピングは以下のパスにJSONとしてバンドルされています:

-   `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## データソース

現在、以下のMITライセンスのリポジトリからマッピングをバンドルしています:

-   `kyle-seongwoo-jun/apple-device-identifiers`

ビルドの決定性を保つため、JSONファイルは特定のアップストリームコミットに固定されています（`apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`に記録されています）。

## データベースの更新

1.  固定したいアップストリームのコミットを選択します（iOS用とmacOS用で1つずつ）。
2.  `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`内のコミットハッシュを更新します。
3.  選択したコミットに固定してJSONファイルを再ダウンロードします:

```
IOS_COMMIT="<ios-device-identifiers.json用のコミットSHA>"
MAC_COMMIT="<mac-device-identifiers.json用のコミットSHA>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4.  `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt`がアップストリームのライセンスと引き続き一致していることを確認します（アップストリームのライセンスが変更された場合は置き換えてください）。
5.  macOSアプリが警告なしにクリーンビルドされることを確認します:

```bash
swift build --package-path apps/macos
```

[RPCアダプター](./rpc.md)[デフォルト AGENTS.md](./AGENTS.default.md)