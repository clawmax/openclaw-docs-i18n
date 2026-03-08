

  浏览器

  
# 浏览器登录

## 手动登录（推荐）

当网站需要登录时，请在**宿主**浏览器配置文件（即 openclaw 浏览器）中**手动登录**。**不要**将您的凭据提供给模型。自动登录通常会触发反机器人防御，并可能导致账户被锁定。返回主浏览器文档：[浏览器](./browser.md)。

## 使用哪个 Chrome 配置文件？

OpenClaw 控制一个**专用的 Chrome 配置文件**（名为 `openclaw`，UI 为橙色色调）。这与您日常使用的浏览器配置文件是分开的。有两种简单的方法可以访问它：

1.  **要求代理打开浏览器**，然后您自己登录。
2.  **通过 CLI 打开它**：

```bash
openclaw browser start
openclaw browser open https://x.com
```

如果您有多个配置文件，请传递 `--browser-profile `（默认为 `openclaw`）。

## X/Twitter：推荐流程

-   **阅读/搜索/查看线程：** 使用**宿主**浏览器（手动登录）。
-   **发布更新：** 使用**宿主**浏览器（手动登录）。

## 沙盒隔离 + 宿主浏览器访问

沙盒化的浏览器会话**更有可能**触发机器人检测。对于 X/Twitter（以及其他严格的网站），建议使用**宿主**浏览器。如果代理处于沙盒中，浏览器工具默认使用沙盒。要允许宿主控制：

```json
{
  agents: {
    defaults: {
      sandbox: {
        mode: "non-main",
        browser: {
          allowHostControl: true,
        },
      },
    },
  },
}
```

然后指定目标为宿主浏览器：

```bash
openclaw browser open https://x.com --browser-profile openclaw --target host
```

或者，为发布更新的代理禁用沙盒隔离。

[浏览器（OpenClaw 托管）](./browser.md)[Chrome 扩展程序](./chrome-extension.md)

---