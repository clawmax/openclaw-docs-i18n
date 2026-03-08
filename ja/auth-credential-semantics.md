

  設定と運用

  
# Auth 認証情報のセマンティクス

このドキュメントは、以下の機能全体で使用される正規の認証情報の適格性および解決セマンティクスを定義します:

-   `resolveAuthProfileOrder`
-   `resolveApiKeyForProfile`
-   `models status --probe`
-   `doctor-auth`

目的は、選択時と実行時の動作を一致させることです。

## 安定した理由コード

-   `ok`
-   `missing_credential`
-   `invalid_expires`
-   `expired`
-   `unresolved_ref`

## トークン認証情報

トークン認証情報 (`type: "token"`) は、インラインの `token` および/または `tokenRef` をサポートします。

### 適格性ルール

1.  `token` と `tokenRef` の両方が存在しない場合、トークンプロファイルは不適格です。
2.  `expires` はオプションです。
3.  `expires` が存在する場合、`0` より大きい有限の数値でなければなりません。
4.  `expires` が無効な場合 (`NaN`、`0`、負の値、非有限、または型が異なる)、プロファイルは `invalid_expires` で不適格です。
5.  `expires` が過去の日時の場合、プロファイルは `expired` で不適格です。
6.  `tokenRef` は `expires` の検証をバイパスしません。

### 解決ルール

1.  解決のセマンティクスは、`expires` に関する適格性のセマンティクスと一致します。
2.  適格なプロファイルの場合、トークン情報はインラインの値または `tokenRef` から解決される可能性があります。
3.  解決不可能な参照は、`models status --probe` の出力で `unresolved_ref` を生成します。

## レガシー互換メッセージング

スクリプト互換性のため、プローブエラーの最初の行は変更されません: `Auth profile credentials are missing or expired.` 人間が理解しやすい詳細と安定した理由コードは、後続の行に追加される場合があります。

[認証](./gateway/authentication.md)[シークレット管理](./gateway/secrets.md)

---