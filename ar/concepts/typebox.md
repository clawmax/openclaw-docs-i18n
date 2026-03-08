title: "مكتبة مخططات TypeBox لبروتوكول Gateway WebSocket"
description: "تعرف على كيفية تعريف TypeBox لبروتوكول Gateway WebSocket للتحقق في وقت التشغيل، وتصدير مخطط JSON، وإنشاء كود Swift. اتبع دليلًا خطوة بخطوة لإضافة طرق جديدة."
keywords: ["typebox", "بروتوكول websocket", "التحقق من صحة المخطط", "مخطط typescript", "إنشاء كود swift", "هندسة البوابة", "مخطط json", "التحقق في وقت التشغيل"]
---

  المفاهيم الداخلية

  
# TypeBox

آخر تحديث: 2026-01-10 TypeBox هي مكتبة مخططات تعتمد على TypeScript أولاً. نستخدمها لتعريف **بروتوكول Gateway WebSocket** (المصافحة، الطلب/الاستجابة، أحداث الخادم). تقود هذه المخططات **التحقق في وقت التشغيل**، و**تصدير مخطط JSON**، و**إنشاء كود Swift** لتطبيق macOS. مصدر واحد للحقيقة؛ كل شيء آخر يتم إنشاؤه. إذا كنت تريد السياق الأعلى مستوى للبروتوكول، ابدأ بـ [هندسة البوابة](./architecture.md).

## النموذج الذهني (30 ثانية)

كل رسالة Gateway WS هي واحدة من ثلاثة إطارات:

-   **طلب**: `{ type: "req", id, method, params }`
-   **استجابة**: `{ type: "res", id, ok, payload | error }`
-   **حدث**: `{ type: "event", event, payload, seq?, stateVersion? }`

يجب أن يكون الإطار الأول طلب `connect`. بعد ذلك، يمكن للعملاء استدعاء طرق (مثل `health`، `send`، `chat.send`) والاشتراك في الأحداث (مثل `presence`، `tick`، `agent`). تدفق الاتصال (الحد الأدنى):

```
العميل                    البوابة
  |---- req:connect -------->|
  |<---- res:hello-ok --------|
  |<---- event:tick ----------|
  |---- req:health ---------->|
  |<---- res:health ----------|
```

الطرق والأحداث الشائعة:

| الفئة | أمثلة | ملاحظات |
| --- | --- | --- |
| الأساسية | `connect`, `health`, `status` | يجب أن يكون `connect` أولاً |
| المراسلة | `send`, `poll`, `agent`, `agent.wait` | التأثيرات الجانبية تحتاج `idempotencyKey` |
| الدردشة | `chat.history`, `chat.send`, `chat.abort`, `chat.inject` | يستخدمها WebChat |
| الجلسات | `sessions.list`, `sessions.patch`, `sessions.delete` | إدارة الجلسات |
| العقد | `node.list`, `node.invoke`, `node.pair.*` | إجراءات Gateway WS + العقد |
| الأحداث | `tick`, `presence`, `agent`, `chat`, `health`, `shutdown` | دفع من الخادم |

توجد القائمة الرسمية في `src/gateway/server.ts` (`METHODS`, `EVENTS`).

## مكان وجود المخططات

-   المصدر: `src/gateway/protocol/schema.ts`
-   أدوات التحقق في وقت التشغيل (AJV): `src/gateway/protocol/index.ts`
-   مصافحة الخادم + توزيع الطرق: `src/gateway/server.ts`
-   عميل العقدة: `src/gateway/client.ts`
-   مخطط JSON المُنشأ: `dist/protocol.schema.json`
-   نماذج Swift المُنشأة: `apps/macos/Sources/OpenClawProtocol/GatewayModels.swift`

## خط الأنابيب الحالي

-   `pnpm protocol:gen`
    -   يكتب مخطط JSON (مسودة‑07) إلى `dist/protocol.schema.json`
-   `pnpm protocol:gen:swift`
    -   يُنشئ نماذج بوابة Swift
-   `pnpm protocol:check`
    -   يشغل كلا المولدين ويتحقق من أن المخرجات مُلتزمة

## كيفية استخدام المخططات في وقت التشغيل

-   **جانب الخادم**: يتم التحقق من كل إطار وارد باستخدام AJV. تقبل المصافحة فقط طلب `connect` تتطابق معلماته مع `ConnectParams`.
-   **جانب العميل**: يتحقق عميل JS من إطارات الأحداث والاستجابة قبل استخدامها.
-   **سطح الطرق**: تعلن البوابة عن `methods` و `events` المدعومة في `hello-ok`.

## أمثلة على الإطارات

الاتصال (الرسالة الأولى):

```json
{
  "type": "req",
  "id": "c1",
  "method": "connect",
  "params": {
    "minProtocol": 2,
    "maxProtocol": 2,
    "client": {
      "id": "openclaw-macos",
      "displayName": "macos",
      "version": "1.0.0",
      "platform": "macos 15.1",
      "mode": "ui",
      "instanceId": "A1B2"
    }
  }
}
```

استجابة hello-ok:

```json
{
  "type": "res",
  "id": "c1",
  "ok": true,
  "payload": {
    "type": "hello-ok",
    "protocol": 2,
    "server": { "version": "dev", "connId": "ws-1" },
    "features": { "methods": ["health"], "events": ["tick"] },
    "snapshot": {
      "presence": [],
      "health": {},
      "stateVersion": { "presence": 0, "health": 0 },
      "uptimeMs": 0
    },
    "policy": { "maxPayload": 1048576, "maxBufferedBytes": 1048576, "tickIntervalMs": 30000 }
  }
}
```

طلب + استجابة:

```json
{ "type": "req", "id": "r1", "method": "health" }
```

```json
{ "type": "res", "id": "r1", "ok": true, "payload": { "ok": true } }
```

حدث:

```json
{ "type": "event", "event": "tick", "payload": { "ts": 1730000000 }, "seq": 12 }
```

## عميل بسيط (Node.js)

أصغر تدفق مفيد: الاتصال + الحالة الصحية.

```typescript
import { WebSocket } from "ws";

const ws = new WebSocket("ws://127.0.0.1:18789");

ws.on("open", () => {
  ws.send(
    JSON.stringify({
      type: "req",
      id: "c1",
      method: "connect",
      params: {
        minProtocol: 3,
        maxProtocol: 3,
        client: {
          id: "cli",
          displayName: "example",
          version: "dev",
          platform: "node",
          mode: "cli",
        },
      },
    }),
  );
});

ws.on("message", (data) => {
  const msg = JSON.parse(String(data));
  if (msg.type === "res" && msg.id === "c1" && msg.ok) {
    ws.send(JSON.stringify({ type: "req", id: "h1", method: "health" }));
  }
  if (msg.type === "res" && msg.id === "h1") {
    console.log("health:", msg.payload);
    ws.close();
  }
});
```

## مثال عملي: إضافة طريقة من البداية للنهاية

مثال: إضافة طلب جديد `system.echo` يعيد `{ ok: true, text }`.

1.  **المخطط (مصدر الحقيقة)**

أضف إلى `src/gateway/protocol/schema.ts`:

```bash
export const SystemEchoParamsSchema = Type.Object(
  { text: NonEmptyString },
  { additionalProperties: false },
);

export const SystemEchoResultSchema = Type.Object(
  { ok: Type.Boolean(), text: NonEmptyString },
  { additionalProperties: false },
);
```

أضف كليهما إلى `ProtocolSchemas` وقم بتصدير الأنواع:

```yaml
SystemEchoParams: SystemEchoParamsSchema,
  SystemEchoResult: SystemEchoResultSchema,
```

```bash
export type SystemEchoParams = Static<typeof SystemEchoParamsSchema>;
export type SystemEchoResult = Static<typeof SystemEchoResultSchema>;
```

2.  **التحقق**

في `src/gateway/protocol/index.ts`، قم بتصدير أداة تحقق AJV:

```bash
export const validateSystemEchoParams = ajv.compile<SystemEchoParams>(SystemEchoParamsSchema);
```

3.  **سلوك الخادم**

أضف معالجًا في `src/gateway/server-methods/system.ts`:

```bash
export const systemHandlers: GatewayRequestHandlers = {
  "system.echo": ({ params, respond }) => {
    const text = String(params.text ?? "");
    respond(true, { ok: true, text });
  },
};
```

سجله في `src/gateway/server-methods.ts` (يدمج بالفعل `systemHandlers`)، ثم أضف `"system.echo"` إلى `METHODS` في `src/gateway/server.ts`.

4.  **إعادة التوليد**

```bash
pnpm protocol:check
```

5.  **الاختبارات + الوثائق**

أضف اختبار خادم في `src/gateway/server.*.test.ts` واذكر الطريقة في الوثائق.

## سلوك إنشاء كود Swift

ينشئ مولد Swift:

-   تعداد `GatewayFrame` مع حالات `req`، `res`، `event`، و `unknown`
-   هياكل/تعدادات حمولة ذات أنواع قوية
-   قيم `ErrorCode` و `GATEWAY_PROTOCOL_VERSION`

يتم الاحتفاظ بأنواع الإطارات غير المعروفة كحمولات خام لتوافقية مستقبلية.

## إصدار البروتوكول + التوافقية

-   `PROTOCOL_VERSION` موجود في `src/gateway/protocol/schema.ts`.
-   يرسل العملاء `minProtocol` + `maxProtocol`؛ يرفض الخادم التطابقات غير المتطابقة.
-   تحتفظ نماذج Swift بأنواع الإطارات غير المعروفة لتجنب كسر العملاء الأقدم.

## أنماط واتفاقيات المخططات

-   تستخدم معظم الكائنات `additionalProperties: false` للحمولات الصارمة.
-   `NonEmptyString` هو الافتراضي لمعرفات وأسماء الطرق/الأحداث.
-   يستخدم المستوى الأعلى `GatewayFrame` **مُميزًا** على `type`.
-   تتطلب الطرق ذات التأثيرات الجانبية عادةً `idempotencyKey` في المعلمات (مثال: `send`، `poll`، `agent`، `chat.send`).
-   يقبل `agent` `internalEvents` اختيارية لسياق التنسيق المُنشأ في وقت التشغيل (على سبيل المثال تسليم إكمال مهمة وكيل فرعي/مجدول)؛ عالج هذا كسطح API داخلي.

## مخطط JSON الحي

يوجد مخطط JSON المُنشأ في المستودع في `dist/protocol.schema.json`. عادةً ما يكون الملف الخام المنشور متاحًا في:

-   [https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json](https://raw.githubusercontent.com/openclaw/openclaw/main/dist/protocol.schema.json)

## عند تغيير المخططات

1.  قم بتحديث مخططات TypeBox.
2.  شغل `pnpm protocol:check`.
3.  قم بإيداع المخطط + نماذج Swift المُعاد توليدها.

[التاريخ والوقت](../date-time.md)[تنسيق Markdown](./markdown-formatting.md)