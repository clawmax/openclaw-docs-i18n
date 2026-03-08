title: "Настройка моделей Amazon Bedrock в шлюзе ИИ OpenClaw"
description: "Узнайте, как настроить и использовать модели Amazon Bedrock с OpenClaw через учетные данные AWS, автоматическое обнаружение моделей и роли экземпляра EC2."
keywords: ["amazon bedrock", "openclaw", "учетные данные aws", "обнаружение моделей", "bedrock converse", "роль iam ec2", "claude bedrock", "шлюз ии"]
---

  Провайдеры

  
# Amazon Bedrock

OpenClaw может использовать модели **Amazon Bedrock** через потоковый провайдер **Bedrock Converse** от pi‑ai. Аутентификация в Bedrock использует **цепочку учетных данных по умолчанию AWS SDK**, а не API-ключ.

## Что поддерживает pi‑ai

-   Провайдер: `amazon-bedrock`
-   API: `bedrock-converse-stream`
-   Аутентификация: Учетные данные AWS (переменные окружения, общий конфиг или роль экземпляра)
-   Регион: `AWS_REGION` или `AWS_DEFAULT_REGION` (по умолчанию: `us-east-1`)

## Автоматическое обнаружение моделей

Если обнаружены учетные данные AWS, OpenClaw может автоматически находить модели Bedrock, поддерживающие **потоковую передачу** и **текстовый вывод**. Обнаружение использует `bedrock:ListFoundationModels` и кэшируется (по умолчанию: 1 час). Параметры конфигурации находятся в `models.bedrockDiscovery`:

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

Примечания:

-   `enabled` по умолчанию `true`, когда присутствуют учетные данные AWS.
-   `region` по умолчанию берется из `AWS_REGION` или `AWS_DEFAULT_REGION`, затем `us-east-1`.
-   `providerFilter` сопоставляет имена провайдеров Bedrock (например, `anthropic`).
-   `refreshInterval` указывается в секундах; установите `0` для отключения кэширования.
-   `defaultContextWindow` (по умолчанию: `32000`) и `defaultMaxTokens` (по умолчанию: `4096`) используются для обнаруженных моделей (переопределите, если знаете лимиты вашей модели).

## Начало работы

1.  Убедитесь, что учетные данные AWS доступны на **хосте шлюза**:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# Опционально:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# Опционально (API-ключ/токен Bearer для Bedrock):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2.  Добавьте провайдера Bedrock и модель в ваш конфиг (ключ `apiKey` не требуется):

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

## Роли экземпляра EC2

При запуске OpenClaw на экземпляре EC2 с прикрепленной ролью IAM, AWS SDK автоматически будет использовать службу метаданных экземпляра (IMDS) для аутентификации. Однако текущая проверка учетных данных в OpenClaw ищет только переменные окружения, а не учетные данные IMDS. **Обходное решение:** Установите `AWS_PROFILE=default`, чтобы указать на наличие учетных данных AWS. Фактическая аутентификация по-прежнему будет использовать роль экземпляра через IMDS.

```bash
# Добавьте в ~/.bashrc или профиль вашей оболочки
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**Необходимые разрешения IAM** для роли экземпляра EC2:

-   `bedrock:InvokeModel`
-   `bedrock:InvokeModelWithResponseStream`
-   `bedrock:ListFoundationModels` (для автоматического обнаружения)

Или прикрепите управляемую политику `AmazonBedrockFullAccess`.

## Быстрая настройка (путь AWS)

```bash
# 1. Создайте роль IAM и профиль экземпляра
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

# 2. Прикрепите к вашему экземпляру EC2
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. На экземпляре EC2 включите обнаружение
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. Установите обходные переменные окружения
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. Убедитесь, что модели обнаружены
openclaw models list
```

## Примечания

-   Для работы с Bedrock требуется **включить доступ к модели** в вашем аккаунте/регионе AWS.
-   Автоматическое обнаружение требует разрешения `bedrock:ListFoundationModels`.
-   Если вы используете профили, установите `AWS_PROFILE` на хосте шлюза.
-   OpenClaw проверяет источник учетных данных в следующем порядке: `AWS_BEARER_TOKEN_BEDROCK`, затем `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`, затем `AWS_PROFILE`, затем цепочка AWS SDK по умолчанию.
-   Поддержка рассуждений зависит от модели; проверьте карточку модели Bedrock для актуальных возможностей.
-   Если вы предпочитаете управляемый ключ, вы также можете разместить совместимый с OpenAI прокси перед Bedrock и настроить его как провайдера OpenAI.

[Anthropic](./anthropic.md)[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)