

  内置工具

  
# apply_patch 工具

使用结构化的补丁格式应用文件更改。这对于多文件或多块（hunk）编辑非常理想，因为单个 `edit` 调用可能不够健壮。该工具接受一个包含一个或多个文件操作的 `input` 字符串：

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

## 参数

-   `input` (必需): 完整的补丁内容，包括 `*** Begin Patch` 和 `*** End Patch`。

## 注意事项

-   补丁路径支持相对路径（相对于工作区目录）和绝对路径。
-   `tools.exec.applyPatch.workspaceOnly` 默认为 `true`（仅限于工作区内）。仅在您有意让 `apply_patch` 在工作区目录之外写入/删除文件时，才将其设置为 `false`。
-   在 `*** Update File:` 块内使用 `*** Move to:` 来重命名文件。
-   需要时，`*** End of File` 标记仅用于文件末尾的插入。
-   实验性功能，默认禁用。通过 `tools.exec.applyPatch.enabled` 启用。
-   仅限 OpenAI（包括 OpenAI Codex）。可选地通过 `tools.exec.applyPatch.allowModels` 按模型进行限制。
-   配置仅在 `tools.exec` 下。

## 示例

```json
{
  "tool": "apply_patch",
  "input": "*** Begin Patch\n*** Update File: src/index.ts\n@@\n-const foo = 1\n+const foo = 2\n*** End Patch"
}
```

[工具](../tools.md)[Brave 搜索](../brave-search.md)