

  RPC و API

  
# قاعدة بيانات نماذج الأجهزة

يعرض تطبيق macOS المرافق أسماء نماذج أجهزة Apple مألوفة في واجهة مستخدم **النسخ** من خلال ربط مُعرّفات نماذج Apple (مثل `iPad16,6`، `Mac16,6`) بأسماء قابلة للقراءة. يتم توفير التعيين كملف JSON تحت:

-   `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## مصدر البيانات

نحن حاليًا نورد التعيين من المستودع المرخص بترخيص MIT:

-   `kyle-seongwoo-jun/apple-device-identifiers`

لضمان قابلية التحديد في عمليات البناء، يتم تثبيت ملفات JSON على عمليات commit محددة من المصدر (مسجلة في `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`).

## تحديث قاعدة البيانات

1.  اختر عمليات commit من المصدر التي تريد التثبيت عليها (واحدة لـ iOS، وواحدة لـ macOS).
2.  قم بتحديث تجزئة الـ commit في `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md`.
3.  أعد تنزيل ملفات JSON، مثبتة على تلك الـ commits:

```
IOS_COMMIT="<commit sha for ios-device-identifiers.json>"
MAC_COMMIT="<commit sha for mac-device-identifiers.json>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4.  تأكد من أن `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` لا يزال مطابقًا للمصدر (استبدله إذا تغيرت الرخصة في المصدر).
5.  تحقق من أن تطبيق macOS يُبنى بدون أخطاء (بدون تحذيرات):

```bash
swift build --package-path apps/macos
```

[RPC Adapters](./rpc.md)[Default AGENTS.md](./AGENTS.default.md)