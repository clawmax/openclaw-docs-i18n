

  المزودون

  
# Amazon Bedrock

يمكن لـ OpenClaw استخدام نماذج **Amazon Bedrock** عبر موفر البث **Bedrock Converse** الخاص بـ pi‑ai. يستخدم مصادقة Bedrock **سلسلة بيانات الاعتماد الافتراضية لـ AWS SDK**، وليس مفتاح API.

## ما يدعمه pi‑ai

-   المزود: `amazon-bedrock`
-   API: `bedrock-converse-stream`
-   المصادقة: بيانات اعتماد AWS (متغيرات البيئة، التكوين المشترك، أو دور المثيل)
-   المنطقة: `AWS_REGION` أو `AWS_DEFAULT_REGION` (الافتراضي: `us-east-1`)

## اكتشاف النماذج التلقائي

إذا تم اكتشاف بيانات اعتماد AWS، يمكن لـ OpenClaw اكتشاف نماذج Bedrock التي تدعم **البث** و**إخراج النص** تلقائيًا. يستخدم الاكتشاف `bedrock:ListFoundationModels` ويتم تخزينه مؤقتًا (الافتراضي: ساعة واحدة). خيارات التكوين موجودة تحت `models.bedrockDiscovery`:

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

ملاحظات:

-   `enabled` تكون `true` افتراضيًا عند وجود بيانات اعتماد AWS.
-   `region` تكون `AWS_REGION` أو `AWS_DEFAULT_REGION` افتراضيًا، ثم `us-east-1`.
-   `providerFilter` تطابق أسماء مزودي Bedrock (على سبيل المثال `anthropic`).
-   `refreshInterval` بالثواني؛ اضبط على `0` لتعطيل التخزين المؤقت.
-   `defaultContextWindow` (الافتراضي: `32000`) و `defaultMaxTokens` (الافتراضي: `4096`) تُستخدم للنماذج المكتشفة (تجاوزها إذا كنت تعرف حدود نموذجك).

## بدء الاستخدام

1.  تأكد من توفر بيانات اعتماد AWS على **مضيف البوابة**:

```bash
export AWS_ACCESS_KEY_ID="AKIA..."
export AWS_SECRET_ACCESS_KEY="..."
export AWS_REGION="us-east-1"
# اختياري:
export AWS_SESSION_TOKEN="..."
export AWS_PROFILE="your-profile"
# اختياري (مفتاح API Bedrock / رمز Bearer):
export AWS_BEARER_TOKEN_BEDROCK="..."
```

2.  أضف مزود Bedrock ونموذجًا إلى تكوينك (لا حاجة إلى `apiKey`):

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

## أدوار مثيل EC2

عند تشغيل OpenClaw على مثيل EC2 مع دور IAM مرفق، سيقوم AWS SDK تلقائيًا باستخدام خدمة بيانات تعريف المثيل (IMDS) للمصادقة. ومع ذلك، فإن اكتشاف بيانات الاعتماد في OpenClaw حاليًا يتحقق فقط من متغيرات البيئة، وليس بيانات اعتماد IMDS. **الحل البديل:** اضبط `AWS_PROFILE=default` للإشارة إلى توفر بيانات اعتماد AWS. تظل المصادقة الفعلية تستخدم دور المثيل عبر IMDS.

```bash
# أضف إلى ~/.bashrc أو ملف تعريف shell الخاص بك
export AWS_PROFILE=default
export AWS_REGION=us-east-1
```

**أذونات IAM المطلوبة** لدور مثيل EC2:

-   `bedrock:InvokeModel`
-   `bedrock:InvokeModelWithResponseStream`
-   `bedrock:ListFoundationModels` (لاكتشاف النماذج التلقائي)

أو قم بإرفاق سياسة الإدارة `AmazonBedrockFullAccess`.

## الإعداد السريع (مسار AWS)

```bash
# 1. إنشاء دور IAM وملف تعريف مثيل
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

# 2. إرفاقه بمثيل EC2 الخاص بك
aws ec2 associate-iam-instance-profile \
  --instance-id i-xxxxx \
  --iam-instance-profile Name=EC2-Bedrock-Access

# 3. على مثيل EC2، تمكين الاكتشاف
openclaw config set models.bedrockDiscovery.enabled true
openclaw config set models.bedrockDiscovery.region us-east-1

# 4. تعيين متغيرات البيئة للحل البديل
echo 'export AWS_PROFILE=default' >> ~/.bashrc
echo 'export AWS_REGION=us-east-1' >> ~/.bashrc
source ~/.bashrc

# 5. التحقق من اكتشاف النماذج
openclaw models list
```

## ملاحظات

-   يتطلب Bedrock **تمكين وصول النموذج** في حساب/منطقة AWS الخاصة بك.
-   يحتاج الاكتشاف التلقائي إلى إذن `bedrock:ListFoundationModels`.
-   إذا كنت تستخدم ملفات تعريف، اضبط `AWS_PROFILE` على مضيف البوابة.
-   يعرض OpenClaw مصدر بيانات الاعتماد بهذا الترتيب: `AWS_BEARER_TOKEN_BEDROCK`، ثم `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY`، ثم `AWS_PROFILE`، ثم سلسلة AWS SDK الافتراضية.
-   يعتمد دعم التفكير على النموذج؛ تحقق من بطاقة نموذج Bedrock للحصول على القدرات الحالية.
-   إذا كنت تفضل تدفقًا يدير المفاتيح، يمكنك أيضًا وضع وكيل متوافق مع OpenAI أمام Bedrock وتكوينه كمزود OpenAI بدلاً من ذلك.

[Anthropic](./anthropic.md)[Cloudflare AI Gateway](./cloudflare-ai-gateway.md)