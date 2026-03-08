title: "OpenClaw AIゲートウェイでAmazon Bedrockモデルを設定する"
description: "AWS認証情報、自動モデル検出、EC2インスタンスロールを使用して、OpenClawでAmazon Bedrockモデルをセットアップおよび使用する方法を学びます。"
keywords: ["amazon bedrock", "openclaw", "aws認証情報", "モデル検出", "bedrock converse", "ec2 iamロール", "claude bedrock", "aiゲートウェイ"]
---

  プロバイダー

  
# Amazon Bedrock

OpenClawは、pi‑aiの**Bedrock Converse**ストリーミングプロバイダーを介して**Amazon Bedrock**モデルを使用できます。Bedrockの認証はAPIキーではなく、**AWS SDKデフォルト認証情報チェーン**を使用します。

## pi‑aiがサポートするもの

-   プロバイダー: `amazon-bedrock`
-   API: `bedrock-converse-stream`
-   認証: AWS認証情報（環境変数、共有設定、またはインスタンスロール）
-   リージョン: `AWS_REGION` または `AWS_DEFAULT_REGION` (デフォルト: `us-east-1`)

## 自動モデル検出

AWS認証情報が検出されると、OpenClawは**ストリーミング**と**テキスト出力**をサポートするBedrockモデルを自動的に検出できます。検出には`bedrock:ListFoundationModels`を使用し、キャッシュされます（デフォルト: 1時間）。設定オプションは`models.bedrockDiscovery`の下にあります:

```json
{
  models: {
    bedrockDiscovery: {
      enabled: true,
      region: "us-east-1",
      providerFilter: ["anthropic", "amazon"],
      refreshInterval: 3600,
      defaultContextWindow: 32000,
      defaultMaxTokens: 4096,
    },
  },
}
```

注意点:

-   `enabled`は、AWS認証情報が存在する場合、デフォルトで`true`になります。
-   `region`は、`AWS_REGION`または`AWS_DEFAULT_REGION`、次に`us-east-1`をデフォルトとします。
-   `providerFilter`はBedrockプロバイダー名（例: `anthropic`）と一致します。
-   `refreshInterval`は秒単位です。`0`に設定するとキャッシュが無効になります。
-   `defaultContextWindow`（デフォルト: `32000`）と`defaultMaxTokens`（デフォルト: `4096`）は、検出されたモデルに使用されます（モデルの制限がわかっている場合は上書きしてください）。

## オンボーディング

1.  **ゲートウェイホスト**でAWS認証情報が利用可能であることを確認します:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# オプション:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# オプション (Bedrock APIキー/ベアラートークン):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2.  設定にBedrockプロバイダーとモデルを追加します（`apiKey`は不要）:

```json
{
  models: {
    providers: {
      "amazon-bedrock": {
        baseUrl: "https://bedrock-runtime.us-east-1.amazonaws.com",
        api: "bedrock-converse-stream",
        auth: "aws-sdk",
        models: [
          {
            id: "us.anthropic.claude-opus-4-6-v1:0",
            name: "Claude Opus 4.6 (Bedrock)",
            reasoning: true,
            input: ["text", "image"],
            cost: { input: 0, output: 0, cacheRead: 0, cacheWrite: 0 },
            contextWindow: 200000,
            maxTokens: 8192,
          },
        ],
      },
    },
  },
  agents: {
    defaults: {
      model: { primary: "amazon-bedrock/us.anthropic.claude-opus-4-6-v1:0" },
    },
  },
}
```

## EC2インスタンスロール

IAMロールがアタッチされたEC2インスタンスでOpenClawを実行する場合、AWS SDKは認証にインスタンスメタデータサービス（IMDS）を自動的に使用します。ただし、OpenClawの認証情報検出は現在、環境変数のみをチェックし、IMDS認証情報はチェックしません。**回避策:** `AWS_PROFILE=default`を設定して、AWS認証情報が利用可能であることを通知します。実際の認証は、IMDSを介したインスタンスロールを使用します。

```bash
# ~/.bashrc またはシェルプロファイルに追加
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

EC2インスタンスロールに必要な**IAM権限**:

-   `bedrock:InvokeModel`
-   `bedrock:InvokeModelWithResponseStream`
-   `bedrock:ListFoundationModels`（自動検出用）

または、管理ポリシー`AmazonBedrockFullAccess`をアタッチします。

## クイックセットアップ（AWSパス）

```bash
# 1. IAMロールとインスタンスプロファイルを作成
aws iam create-role --role-name EC2-Bedrock-Access \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {"Service": "ec2.amazonaws.com"},
      "Action": "sts:AssumeRole"
    }]
  }'

aws iam attach-role-policy --role-name EC2-Bedrock-Access \
  --policy-arn arn:aws:iam::aws:policy/AmazonBedrockFullAccess

aws iam create-instance-profile --instance-profile-name EC2-Bedrock-Access
aws iam add-role-to-instance-profile \
  --instance-profile-name EC2-Bedrock-Access \
  --role-name EC2-Bedrock-Access

# 2. EC2インスタンスにアタッチ
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. EC2インスタンス上で、検出を有効化
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. 回避策の環境変数を設定
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. モデルが検出されることを確認
openclaw models list
```

## 注意点

-   Bedrockでは、AWSアカウント/リージョンで**モデルアクセス**が有効になっている必要があります。
-   自動検出には`bedrock:ListFoundationModels`権限が必要です。
-   プロファイルを使用する場合は、ゲートウェイホストで`AWS_PROFILE`を設定してください。
-   OpenClawは認証情報ソースを次の順序で表面化します: `AWS_BEARER_TOKEN_BEDROCK`、次に`AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`、次に`AWS_PROFILE`、次にデフォルトのAWS SDKチェーン。
-   推論サポートはモデルに依存します。現在の機能についてはBedrockモデルカードを確認してください。
-   管理されたキーフローを希望する場合は、Bedrockの前にOpenAI互換プロキシを配置し、それをOpenAIプロバイダーとして設定することもできます。

[Anthropic](./anthropic.md)[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)