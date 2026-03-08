

  CLI コマンド

  
# devices

デバイスペアリングリクエストとデバイススコープトークンを管理します。

## コマンド

### openclaw devices list

保留中のペアリングリクエストとペアリング済みデバイスを一覧表示します。

```bash
openclaw devices list
openclaw devices list --json
```

### openclaw devices remove &lt;deviceId&gt;

ペアリング済みデバイスエントリを1つ削除します。

```bash
openclaw devices remove <deviceId>
openclaw devices remove <deviceId> --json
```

### openclaw devices clear --yes \[--pending\]

ペアリング済みデバイスを一括削除します。

```bash
openclaw devices clear --yes
openclaw devices clear --yes --pending
openclaw devices clear --yes --pending --json
```

### openclaw devices approve \[requestId\] \[--latest\]

保留中のデバイスペアリングリクエストを承認します。`requestId` が省略された場合、OpenClaw は自動的に最新の保留中リクエストを承認します。

```bash
openclaw devices approve
openclaw devices approve <requestId>
openclaw devices approve --latest
```

### openclaw devices reject &lt;requestId&gt;

保留中のデバイスペアリングリクエストを拒否します。

```bash
openclaw devices reject <requestId>
```

### openclaw devices rotate --device &lt;id&gt; --role &lt;role&gt; \[--scope &lt;scope...&gt;\]

特定のロール（オプションでスコープを更新）のデバイストークンをローテーションします。

```bash
openclaw devices rotate --device <deviceId> --role operator --scope operator.read --scope operator.write
```

### openclaw devices revoke --device &lt;id&gt; --role &lt;role&gt;

特定のロールのデバイストークンを失効させます。

```bash
openclaw devices revoke --device <deviceId> --role node
```

## 共通オプション

-   `--url `: ゲートウェイ WebSocket URL（設定されている場合はデフォルトで `gateway.remote.url`）。
-   `--token `: ゲートウェイトークン（必要な場合）。
-   `--password `: ゲートウェイパスワード（パスワード認証）。
-   `--timeout `: RPC タイムアウト。
-   `--json`: JSON 出力（スクリプトでの使用を推奨）。

注: `--url` を設定した場合、CLI は設定ファイルや環境変数の認証情報にフォールバックしません。`--token` または `--password` を明示的に渡してください。明示的な認証情報がない場合はエラーになります。

## 注意事項

-   トークンローテーションは新しいトークン（機密情報）を返します。シークレットとして扱ってください。
-   これらのコマンドには `operator.pairing`（または `operator.admin`）スコープが必要です。
-   `devices clear` は意図的に `--yes` で保護されています。
-   ローカルループバックでペアリングスコープが利用できない場合（かつ明示的な `--url` が渡されていない場合）、list/approve はローカルペアリングのフォールバックを使用できます。

[ダッシュボード](./dashboard.md)[ディレクトリ](./directory.md)