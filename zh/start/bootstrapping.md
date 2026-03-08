

  引导

  
# 代理引导

引导是**首次运行**的初始化流程，用于准备代理工作区并收集身份详细信息。它发生在入门之后，即代理首次启动时。

## 引导的作用

在代理首次运行时，OpenClaw 会引导工作区（默认为 `~/.openclaw/workspace`）：

-   创建种子文件 `AGENTS.md`、`BOOTSTRAP.md`、`IDENTITY.md`、`USER.md`。
-   运行简短的问答流程（一次一个问题）。
-   将身份信息和偏好设置写入 `IDENTITY.md`、`USER.md`、`SOUL.md`。
-   完成后移除 `BOOTSTRAP.md`，确保该流程仅运行一次。

## 运行位置

引导始终在**网关主机**上运行。如果 macOS 应用程序连接到远程网关，则工作区和引导文件将位于该远程机器上。

> **ℹ️** 当网关在另一台机器上运行时，请在网关主机上编辑工作区文件（例如，`user@gateway-host:~/.openclaw/workspace`）。

## 相关文档

-   macOS 应用程序入门：[入门指南](./onboarding.md)
-   工作区布局：[代理工作区](../concepts/agent-workspace.md)

[OAuth](../concepts/oauth.md)[会话管理](../concepts/session.md)