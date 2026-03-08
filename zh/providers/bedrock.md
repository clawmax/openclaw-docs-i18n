

  提供商

  
# Amazon Bedrock

OpenClaw 可以通过 pi‑ai 的 **Bedrock Converse** 流式提供商使用 **Amazon Bedrock** 模型。Bedrock 身份验证使用 **AWS SDK 默认凭证链**，而非 API 密钥。

## pi‑ai 支持的功能

-   提供商：`amazon-bedrock`
-   API：`bedrock-converse-stream`
-   身份验证：AWS 凭证（环境变量、共享配置或实例角色）
-   区域：`AWS_REGION` 或 `AWS_DEFAULT_REGION`（默认：`us-east-1`）

## 自动模型发现

如果检测到 AWS 凭证，OpenClaw 可以自动发现支持**流式传输**和**文本输出**的 Bedrock 模型。发现过程使用 `bedrock:ListFoundationModels` 并会被缓存（默认：1 小时）。配置选项位于 `models.bedrockDiscovery` 下：

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

注意事项：

-   当存在 AWS 凭证时，`enabled` 默认为 `true`。
-   `region` 默认为 `AWS_REGION` 或 `AWS_DEFAULT_REGION`，然后是 `us-east-1`。
-   `providerFilter` 匹配 Bedrock 提供商名称（例如 `anthropic`）。
-   `refreshInterval` 单位为秒；设置为 `0` 可禁用缓存。
-   `defaultContextWindow`（默认：`32000`）和 `defaultMaxTokens`（默认：`4096`）用于发现的模型（如果您知道模型限制，可以覆盖这些值）。

## 入门指南

1.  确保 **网关主机** 上可用的 AWS 凭证：

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# 可选：
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# 可选（Bedrock API 密钥/承载令牌）：
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2.  将 Bedrock 提供商和模型添加到您的配置中（无需 `apiKey`）：

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

## EC2 实例角色

当 OpenClaw 运行在附加了 IAM 角色的 EC2 实例上时，AWS SDK 将自动使用实例元数据服务（IMDS）进行身份验证。但是，OpenClaw 的凭证检测目前仅检查环境变量，而不检查 IMDS 凭证。**解决方法：** 设置 `AWS_PROFILE=default` 以表示 AWS 凭证可用。实际的身份验证仍然通过 IMDS 使用实例角色。

```bash
# 添加到 ~/.bashrc 或您的 shell 配置文件中
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

EC2 实例角色所需的 **IAM 权限**：

-   `bedrock:InvokeModel`
-   `bedrock:InvokeModelWithResponseStream`
-   `bedrock:ListFoundationModels`（用于自动发现）

或者附加托管策略 `AmazonBedrockFullAccess`。

## 快速设置（AWS 路径）

```bash
# 1. 创建 IAM 角色和实例配置文件
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

# 2. 附加到您的 EC2 实例
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. 在 EC2 实例上，启用发现功能
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. 设置解决方法环境变量
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. 验证模型是否被发现
openclaw models list
```

## 注意事项

-   Bedrock 要求您的 AWS 账户/区域已启用**模型访问**。
-   自动发现需要 `bedrock:ListFoundationModels` 权限。
-   如果您使用配置文件，请在网关主机上设置 `AWS_PROFILE`。
-   OpenClaw 按以下顺序暴露凭证来源：`AWS_BEARER_TOKEN_BEDROCK`，然后是 `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`，接着是 `AWS_PROFILE`，最后是默认的 AWS SDK 链。
-   推理支持取决于模型；请查看 Bedrock 模型卡片了解当前能力。
-   如果您更喜欢托管密钥流程，也可以在 Bedrock 前面放置一个 OpenAI 兼容的代理，并将其配置为 OpenAI 提供商。

[Anthropic](./anthropic.md)[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)