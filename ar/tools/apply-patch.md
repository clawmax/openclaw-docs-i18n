

  أدوات مدمجة

  
# أداة apply_patch

قم بتطبيق تغييرات الملفات باستخدام تنسيق تصحيح منظم. هذه الأداة مثالية للتعديلات متعددة الملفات أو متعددة الأجزاء حيث أن استدعاء `edit` واحد سيكون هشًا. تقبل الأداة سلسلة `input` واحدة تغلف عملية واحدة أو أكثر على الملفات:

```markdown
*** Begin Patch
*** Add File: path/to/file.txt
+line 1
+line 2
*** Update File: src/app.ts
@@
-old line
+new line
*** Delete File: obsolete.txt
*** End Patch
```

## المعاملات

-   `input` (مطلوب): محتويات التصحيح الكاملة بما في ذلك `*** Begin Patch` و `*** End Patch`.

## ملاحظات

-   تدعم مسارات التصحيح المسارات النسبية (من دليل مساحة العمل) والمسارات المطلقة.
-   `tools.exec.applyPatch.workspaceOnly` تكون `true` بشكل افتراضي (مقيدة بمساحة العمل). اضبطها على `false` فقط إذا كنت تريد عمدًا أن تكتب `apply_patch` أو تحذف خارج دليل مساحة العمل.
-   استخدم `*** Move to:` داخل جزء `*** Update File:` لإعادة تسمية الملفات.
-   `*** End of File` يحدد إدراجًا عند نهاية الملف فقط عند الحاجة.
-   تجريبية ومعطلة بشكل افتراضي. قم بتمكينها باستخدام `tools.exec.applyPatch.enabled`.
-   حصرية لمنصة OpenAI (بما في ذلك OpenAI Codex). يمكنك تقييدها حسب النموذج عبر `tools.exec.applyPatch.allowModels`.
-   الإعدادات موجودة فقط تحت `tools.exec`.

## مثال

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

[الأدوات](../tools.md)[Brave Search](../brave-search.md)