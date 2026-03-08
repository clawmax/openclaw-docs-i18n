

  ホスティングとデプロイ

  
# Renderにデプロイ

Infrastructure as Codeを使用してRenderにOpenClawをデプロイします。付属の `render.yaml` Blueprintは、サービス、ディスク、環境変数など、スタック全体を宣言的に定義しているため、ワンクリックでデプロイでき、インフラをコードと共にバージョン管理できます。

## 前提条件

-   [Renderアカウント](https://render.com) (無料枠あり)
-   お好みの[モデルプロバイダー](../providers.md)からのAPIキー

## Render Blueprintでデプロイ

[Renderにデプロイ](https://render.com/deploy?repo=https://github.com/openclaw/openclaw) このリンクをクリックすると:

1.  このリポジトリのルートにある `render.yaml` Blueprintから新しいRenderサービスを作成します。
2.  `SETUP_PASSWORD` の設定を促します。
3.  Dockerイメージをビルドしてデプロイします。

デプロイ後、サービスURLは `https://<service-name>.onrender.com` というパターンになります。

## Blueprintの理解

Render Blueprintは、インフラを定義するYAMLファイルです。このリポジトリの `render.yaml` は、OpenClawを実行するために必要なすべてを設定します:

```yaml
services:
  - type: web
    name: openclaw
    runtime: docker
    plan: starter
    healthCheckPath: /health
    envVars:
      - key: PORT
        value: "8080"
      - key: SETUP_PASSWORD
        sync: false # prompts during deploy
      - key: OPENCLAW_STATE_DIR
        value: /data/.openclaw
      - key: OPENCLAW_WORKSPACE_DIR
        value: /data/workspace
      - key: OPENCLAW_GATEWAY_TOKEN
        generateValue: true # auto-generates a secure token
    disk:
      name: openclaw-data
      mountPath: /data
      sizeGB: 1
```

使用されている主なBlueprint機能:

| 機能 | 目的 |
| --- | --- |
| `runtime: docker` | リポジトリのDockerfileからビルド |
| `healthCheckPath` | Renderが `/health` を監視し、異常なインスタンスを再起動 |
| `sync: false` | デプロイ時に値を入力 (シークレット) |
| `generateValue: true` | 暗号的に安全な値を自動生成 |
| `disk` | 再デプロイ後も永続するストレージ |

## プランの選択

| プラン | スピンダウン | ディスク | 最適な用途 |
| --- | --- | --- | --- |
| 無料 | 15分のアイドル後 | 利用不可 | テスト、デモ |
| スターター | なし | 1GB+ | 個人利用、小規模チーム |
| スタンダード+ | なし | 1GB+ | 本番環境、複数チャネル |

Blueprintはデフォルトで `starter` を使用します。無料枠を使用するには、フォークした `render.yaml` で `plan: free` に変更してください (ただし、永続ディスクがないため、デプロイごとに設定がリセットされることに注意)。

## デプロイ後

### セットアップウィザードを完了する

1.  `https://<your-service>.onrender.com/setup` にアクセス
2.  `SETUP_PASSWORD` を入力
3.  モデルプロバイダーを選択し、APIキーを貼り付け
4.  オプションでメッセージングチャネル (Telegram, Discord, Slack) を設定
5.  **セットアップを実行** をクリック

### コントロールUIにアクセス

ウェブダッシュボードは `https://<your-service>.onrender.com/openclaw` で利用可能です。

## Renderダッシュボードの機能

### ログ

**ダッシュボード → 対象サービス → ログ** でリアルタイムログを表示。以下でフィルタリング可能:

-   ビルドログ (Dockerイメージ作成)
-   デプロイログ (サービス起動)
-   ランタイムログ (アプリケーション出力)

### シェルアクセス

デバッグのため、**ダッシュボード → 対象サービス → シェル** からシェルセッションを開けます。永続ディスクは `/data` にマウントされています。

### 環境変数

**ダッシュボード → 対象サービス → 環境** で変数を変更。変更は自動的に再デプロイをトリガーします。

### 自動デプロイ

元のOpenClawリポジトリを使用している場合、RenderはOpenClawを自動デプロイしません。更新するには、ダッシュボードから手動でBlueprint同期を実行してください。

## カスタムドメイン

1.  **ダッシュボード → 対象サービス → 設定 → カスタムドメイン** に移動
2.  ドメインを追加
3.  指示に従ってDNSを設定 (CNAMEを `*.onrender.com` に)
4.  Renderは自動的にTLS証明書をプロビジョニングします

## スケーリング

Renderは水平および垂直スケーリングをサポート:

-   **垂直**: プランを変更してCPU/RAMを増強
-   **水平**: インスタンス数を増加 (スタンダードプラン以上)

OpenClawの場合、通常は垂直スケーリングで十分です。水平スケーリングには、スティッキーセッションまたは外部の状態管理が必要です。

## バックアップと移行

設定とワークスペースはいつでもエクスポート可能:

```
https://<your-service>.onrender.com/setup/export
```

これにより、任意のOpenClawホストで復元可能なポータブルなバックアップがダウンロードされます。

## トラブルシューティング

### サービスが起動しない

Renderダッシュボードのデプロイログを確認。一般的な問題:

-   `SETUP_PASSWORD` の不足 — Blueprintは入力を促しますが、設定されていることを確認
-   ポート不一致 — `PORT=8080` がDockerfileで公開されているポートと一致していることを確認

### コールドスタートが遅い (無料枠)

無料枠のサービスは、15分間の非アクティブ後にスピンダウンします。スピンダウン後の最初のリクエストは、コンテナが起動するまで数秒かかります。常時稼働にはスタータープランにアップグレードしてください。

### 再デプロイ後のデータ損失

これは無料枠 (永続ディスクなし) で発生します。有料プランにアップグレードするか、`/setup/export` 経由で定期的に設定をエクスポートしてください。

### ヘルスチェック失敗

Renderは30秒以内に `/health` から200応答を期待します。ビルドは成功するがデプロイが失敗する場合、サービスの起動に時間がかかりすぎている可能性があります。以下を確認:

-   エラーのためのビルドログ
-   コンテナが `docker build && docker run` でローカルで実行されるかどうか

[Railwayでデプロイ](./railway.md)[Northflankでデプロイ](./northflank.md)