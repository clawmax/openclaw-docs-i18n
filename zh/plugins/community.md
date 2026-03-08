

  扩展

  
# 社区插件

本页面追踪高质量的 **社区维护插件** 以供 OpenClaw 使用。当插件达到质量标准时，我们接受在此处添加社区插件的 PR。

## 上架要求

-   插件包已发布在 npmjs 上（可通过 `openclaw plugins install <npm-spec>` 安装）。
-   源代码托管在 GitHub（公开仓库）。
-   仓库包含设置/使用文档和问题追踪器。
-   插件有明确的维护信号（活跃的维护者、近期更新或及时的问题处理）。

## 如何提交

请提交一个 PR，将您的插件添加到此页面，包含以下信息：

-   插件名称
-   npm 包名
-   GitHub 仓库 URL
-   一行描述
-   安装命令

## 审核标准

我们更倾向于有用、有文档且安全运行的插件。低质量封装、所有权不明确或无人维护的包可能会被拒绝。

## 条目格式

添加条目时请使用此格式：

-   **插件名称** — 简短描述 npm: `@scope/package` repo: `https://github.com/org/repo` install: `openclaw plugins install @scope/package`

## 已列出的插件

-   **WeChat** — 通过 WeChatPadPro（iPad 协议）将 OpenClaw 连接到微信个人账户。支持文本、图片和文件交换，以及关键词触发的对话。 npm: `@icesword760/openclaw-wechat` repo: `https://github.com/icesword0760/openclaw-wechat` install: `openclaw plugins install @icesword760/openclaw-wechat`

[插件](../tools/plugin.md)[语音通话插件](./voice-call.md)

---