

  المساهمة

  
# خط أنابيب التكامل المستمر

يعمل التكامل المستمر عند كل دفع إلى الفرع `main` وكل طلب سحب. يستخدم نطاقًا ذكيًا لتخطي المهام المكلفة عندما تتغير فقط الوثائق أو الكود الأصلي.

## نظرة عامة على المهام

| المهمة | الغرض | وقت التشغيل |
| --- | --- | --- |
| `docs-scope` | اكتشاف التغييرات الخاصة بالوثائق فقط | دائمًا |
| `changed-scope` | اكتشاف المناطق التي تغيرت (node/macos/android/windows) | طلبات السحب غير المتعلقة بالوثائق |
| `check` | أنواع TypeScript، والتدقيق النحوي، والتنسيق | الدفع إلى `main`، أو طلبات السحب ذات التغييرات المتعلقة بـ Node |
| `check-docs` | تدقيق Markdown + فحص الروابط المعطلة | عند تغيير الوثائق |
| `code-analysis` | فحص عتبة سطور الكود (1000 سطر) | طلبات السحب فقط |
| `secrets` | اكتشاف الأسرار المسربة | دائمًا |
| `build-artifacts` | بناء التوزيعة مرة واحدة، ومشاركتها مع المهام الأخرى | تغييرات غير متعلقة بالوثائق، وتغييرات node |
| `release-check` | التحقق من صحة محتويات حزمة npm | بعد البناء |
| `checks` | اختبارات Node/Bun + فحص البروتوكول | تغييرات غير متعلقة بالوثائق، وتغييرات node |
| `checks-windows` | اختبارات خاصة بـ Windows | تغييرات غير متعلقة بالوثائق، وتغييرات متعلقة بـ Windows |
| `macos` | تدقيق/بناء/اختبار Swift + اختبارات TS | طلبات السحب ذات التغييرات المتعلقة بـ macos |
| `android` | بناء Gradle + الاختبارات | تغييرات غير متعلقة بالوثائق، وتغييرات android |

## ترتيب الفشل السريع

يتم ترتيب المهام بحيث تفشل الفحوصات الرخيصة قبل تشغيل المكلفة:

1.  `docs-scope` + `code-analysis` + `check` (متوازية، ~1-2 دقيقة)
2.  `build-artifacts` (معلقة على ما سبق)
3.  `checks`، `checks-windows`، `macos`، `android` (معلقة على البناء)

يوجد منطق النطاق في `scripts/ci-changed-scope.mjs` ويتم تغطيته بواسطة اختبارات الوحدة في `src/scripts/ci-changed-scope.test.ts`.

## المنفذات

| المنفذ | المهام |
| --- | --- |
| `blacksmith-16vcpu-ubuntu-2404` | معظم مهام Linux، بما في ذلك اكتشاف النطاق |
| `blacksmith-32vcpu-windows-2025` | `checks-windows` |
| `macos-latest` | `macos`، `ios` |

## الأوامر المحلية المكافئة

```bash
pnpm check          # types + lint + format
pnpm test           # vitest tests
pnpm check:docs     # docs format + lint + broken links
pnpm release:check  # validate npm pack
```

[سير عمل تطوير Pi](./pi-dev.md)[مراكز الوثائق](./start/hubs.md)