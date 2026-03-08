title: "OpenClaw Zalo 个人插件设置与使用指南"
description: "学习如何安装、配置和使用 OpenClaw 的 Zalo 个人插件，通过 CLI 和智能体工具自动化个人 Zalo 用户账户。"
keywords: ["zalo 个人插件", "openclaw zalouser", "zalo 自动化", "openclaw 插件安装", "zalo 用户账户", "频道配置", "openclaw cli", "zalo 聊天机器人"]
---

  扩展

  
# Zalo 个人插件

通过插件为 OpenClaw 提供 Zalo 个人账户支持，使用原生 `zca-js` 来自动化一个普通的 Zalo 用户账户。

> **警告：** 非官方自动化可能导致账户被暂停或封禁。使用风险自负。

## 命名

频道 ID 为 `zalouser`，以明确表示这是自动化一个**个人 Zalo 用户账户**（非官方）。我们保留 `zalo` 用于未来潜在的官方 Zalo API 集成。

## 运行环境

此插件运行在**网关进程内部**。如果您使用远程网关，请在**运行网关的机器上**安装/配置它，然后重启网关。不需要外部的 `zca`/`openzca` CLI 二进制文件。

## 安装

### 选项 A：从 npm 安装

```bash
openclaw plugins install @openclaw/zalouser
```

之后重启网关。

### 选项 B：从本地文件夹安装（开发）

```bash
openclaw plugins install ./extensions/zalouser
cd ./extensions/zalouser && pnpm install
```

之后重启网关。

## 配置

频道配置位于 `channels.zalouser` 下（而非 `plugins.entries.*`）：

```json
{
  channels: {
    zalouser: {
      enabled: true,
      dmPolicy: "pairing",
    },
  },
}
```

## CLI

```bash
openclaw channels login --channel zalouser
openclaw channels logout --channel zalouser
openclaw channels status --probe
openclaw message send --channel zalouser --target <threadId> --message "Hello from OpenClaw"
openclaw directory peers list --channel zalouser --query "name"
```

## 智能体工具

工具名称：`zalouser` 操作：`send`, `image`, `link`, `friends`, `groups`, `me`, `status` 频道消息操作也支持 `react` 用于消息回应。

[语音通话插件](./voice-call.md)[插件清单](./manifest.md)

---