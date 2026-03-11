

  التطبيق المرافق لنظام macOS

  
# إصدار macOS

يتم شحن هذا التطبيق الآن مع تحديثات Sparkle التلقائية. يجب أن تكون إصدارات التوزيع موقعة بـ Developer ID، ومضغوطة في ملف zip، ونشرها مع إدخال appcast موقّع.

## المتطلبات الأساسية

-   شهادة Developer ID Application مثبتة (مثال: `Developer ID Application:  ()`).
-   مسار مفتاح Sparkle الخاص مضبوط في البيئة كـ `SPARKLE_PRIVATE_KEY_FILE` (المسار إلى مفتاح ed25519 الخاص بـ Sparkle؛ المفتاح العام مضمن في Info.plist). إذا كان مفقودًا، تحقق من `~/.profile`.
-   بيانات اعتماد Notary (ملف تعريف Keychain أو مفتاح API) لـ `xcrun notarytool` إذا كنت تريد توزيع DMG/zip آمن لـ Gatekeeper.
    -   نستخدم ملف تعريف Keychain باسم `openclaw-notary`، تم إنشاؤه من متغيرات بيئة مفتاح API لـ App Store Connect في ملف تعريف shell الخاص بك:
        -   `APP_STORE_CONNECT_API_KEY_P8`, `APP_STORE_CONNECT_KEY_ID`, `APP_STORE_CONNECT_ISSUER_ID`
        -   `echo "$APP_STORE_CONNECT_API_KEY_P8" | sed 's/\\n/\n/g' > /tmp/openclaw-notary.p8`
        -   `xcrun notarytool store-credentials "openclaw-notary" --key /tmp/openclaw-notary.p8 --key-id "$APP_STORE_CONNECT_KEY_ID" --issuer "$APP_STORE_CONNECT_ISSUER_ID"`
-   تبعيات `pnpm` مثبتة (`pnpm install --config.node-linker=hoisted`).
-   يتم جلب أدوات Sparkle تلقائيًا عبر SwiftPM في `apps/macos/.build/artifacts/sparkle/Sparkle/bin/` (`sign_update`, `generate_appcast`, إلخ.).

## البناء والتغليف

ملاحظات:

-   `APP_BUILD` يقابل `CFBundleVersion`/`sparkle:version`؛ حافظ على أن تكون رقمية + رتيبة (بدون `-beta`)، وإلا ستقارنها Sparkle على أنها متساوية.
-   إذا تم حذف `APP_BUILD`، فإن `scripts/package-mac-app.sh` تستنتج قيمة افتراضية آمنة لـ Sparkle من `APP_VERSION` (`YYYYMMDDNN`: الإصدارات المستقرة تستخدم `90` افتراضيًا، والإصدارات التجريبية تستخدم مسارًا مشتقًا من اللاحقة) وتستخدم القيمة الأعلى بين تلك القيمة وعدد عمليات commit في git.
-   لا يزال بإمكانك تجاوز `APP_BUILD` صراحةً عندما تتطلب هندسة الإصدار قيمة رتيبة محددة.
-   الافتراضي هو البنية الحالية (`$(uname -m)`). لإصدارات التوزيع/البنى الشاملة، عيّن `BUILD_ARCHS="arm64 x86_64"` (أو `BUILD_ARCHS=all`).
-   استخدم `scripts/package-mac-dist.sh` لقطع أثرية التوزيع (zip + DMG + الاعتماد). استخدم `scripts/package-mac-app.sh` للتغليف المحلي/للتنمية.

```bash
# من جذر المستودع؛ عيّن معرفات الإصدار حتى يتم تمكين تغذية Sparkle.
# يجب أن يكون APP_BUILD رقميًا + رتيبًا لمقارنة Sparkle.
# الافتراضي هو الاستنتاج التلقائي من APP_VERSION عند حذفه.
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-app.sh

# ضغط للتوزيع (يتضمن resource forks لدعم دلتا Sparkle)
ditto -c -k --sequesterRsrc --keepParent dist/OpenClaw.app dist/OpenClaw-2026.3.7.zip

# اختياري: أيضًا بناء DMG مزخرف للمستخدمين (سحب إلى /Applications)
scripts/create-dmg.sh dist/OpenClaw.app dist/OpenClaw-2026.3.7.dmg

# موصى به: بناء + اعتماد/تثبيت zip + DMG
# أولاً، أنشئ ملف تعريف Keychain مرة واحدة:
#   xcrun notarytool store-credentials "openclaw-notary" \
#     --apple-id "<apple-id>" --team-id "<team-id>" --password "<app-specific-password>"
NOTARIZE=1 NOTARYTOOL_PROFILE=openclaw-notary \
BUNDLE_ID=ai.openclaw.mac \
APP_VERSION=2026.3.7 \
BUILD_CONFIG=release \
SIGN_IDENTITY="Developer ID Application: <Developer Name> (<TEAMID>)" \
scripts/package-mac-dist.sh

# اختياري: شحن dSYM مع الإصدار
ditto -c -k --keepParent apps/macos/.build/release/OpenClaw.app.dSYM dist/OpenClaw-2026.3.7.dSYM.zip
```

## إدخال Appcast

استخدم منشئ ملاحظات الإصدار حتى تعرض Sparkle الملاحظات بتنسيق HTML:

```
SPARKLE_PRIVATE_KEY_FILE=/path/to/ed25519-private-key scripts/make_appcast.sh dist/OpenClaw-2026.3.7.zip https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml
```

ينشئ ملاحظات إصدار HTML من `CHANGELOG.md` (عبر [`scripts/changelog-to-html.sh`](https://github.com/openclaw/openclaw/blob/main/scripts/changelog-to-html.sh)) ويضمنها في إدخال appcast. قم بتثبيت تحديث `appcast.xml` جنبًا إلى جنب مع أصول الإصدار (zip + dSYM) عند النشر.

## النشر والتحقق

-   ارفع `OpenClaw-2026.3.7.zip` (و `OpenClaw-2026.3.7.dSYM.zip`) إلى إصدار GitHub للوسم `v2026.3.7`.
-   تأكد من أن عنوان URL الخام لـ appcast يطابق التغذية المضمنة: `https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml`.
-   فحوصات السلامة:
    -   `curl -I https://raw.githubusercontent.com/openclaw/openclaw/main/appcast.xml` تُرجع 200.
    -   `curl -I ` تُرجع 200 بعد رفع الأصول.
    -   على إصدار عام سابق، شغّل "Check for Updates…" من علامة التبويب About وتحقق من أن Sparkle يثبت الإصدار الجديد بنظافة.

تعريف الإكمال: التطبيق الموقّع و appcast منشوران، وتدفق التحديث يعمل من إصدار مثبت أقدم، وأصول الإصدار مرفقة بإصدار GitHub.

[توقيع macOS](./signing.md)[Gateway على macOS](./bundled-gateway.md)