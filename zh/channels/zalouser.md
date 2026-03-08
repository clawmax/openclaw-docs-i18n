

  消息平台

  
# Zalo 个人账号

状态：实验性。此集成通过 OpenClaw 内部的原生 `zca-js` 自动化**个人 Zalo 账号**。

> **警告：** 这是一个非官方集成，可能导致账号被暂停/封禁。使用风险自负。

## 所需插件

Zalo 个人账号以插件形式提供，不包含在核心安装包中。

-   通过 CLI 安装：`openclaw plugins install @openclaw/zalouser`
-   或从源代码检出安装：`openclaw plugins install ./extensions/zalouser`
-   详情：[插件](../tools/plugin.md)

无需外部 `zca`/`openzca` CLI 二进制文件。

## 快速设置（新手）

1.  安装插件（见上文）。
2.  登录（在网关机器上使用二维码）：
    -   `openclaw channels login --channel zalouser`
    -   使用 Zalo 手机应用扫描二维码。
3.  启用频道：

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

4.  重启网关（或完成引导流程）。
5.  私信访问默认为配对模式；首次联系时批准配对码。

## 这是什么

-   完全通过 `zca-js` 在进程内运行。
-   使用原生事件监听器接收入站消息。
-   直接通过 JS API 发送回复（文本/媒体/链接）。
-   专为 Zalo Bot API 不可用的“个人账号”用例设计。

## 命名

频道 ID 为 `zalouser`，以明确表示此功能自动化的是**个人 Zalo 用户账号**（非官方）。我们保留 `zalo` 用于未来潜在的官方 Zalo API 集成。

## 查找 ID（目录）

使用目录 CLI 来发现联系人/群组及其 ID：

```bash
openclaw directory self --channel zalouser
openclaw directory peers list --channel zalouser --query "name"
openclaw directory groups list --channel zalouser --query "work"
```

## 限制

-   出站文本被分块为约 2000 个字符（Zalo 客户端限制）。
-   默认阻止流式传输。

## 访问控制（私信）

`channels.zalouser.dmPolicy` 支持：`pairing | allowlist | open | disabled`（默认：`pairing`）。`channels.zalouser.allowFrom` 接受用户 ID 或名称。在引导过程中，名称会通过插件进程内的联系人查找解析为 ID。通过以下方式批准：

-   `openclaw pairing list zalouser`
-   `openclaw pairing approve zalouser `

## 群组访问（可选）

-   默认：`channels.zalouser.groupPolicy = "open"`（允许群组）。当未设置时，使用 `channels.defaults.groupPolicy` 覆盖默认值。
-   使用以下方式限制到允许列表：
    -   `channels.zalouser.groupPolicy = "allowlist"`
    -   `channels.zalouser.groups`（键是群组 ID 或名称）
-   阻止所有群组：`channels.zalouser.groupPolicy = "disabled"`。
-   配置向导可以提示输入群组允许列表。
-   启动时，OpenClaw 将允许列表中的群组/用户名称解析为 ID 并记录映射关系；未解析的条目保持原样。

示例：

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "123456789": { allow: true },
        "Work Chat": { allow: true },
      },
    },
  },
}
```

### 群组提及门控

-   `channels.zalouser.groups..requireMention` 控制群组回复是否需要提及。
-   解析顺序：确切的群组 ID/名称 -> 规范化的群组 slug -> `*` -> 默认值 (`true`)。
-   这同时适用于允许列表中的群组和开放群组模式。

示例：

```json
{
  channels: {
    zalouser: {
      groupPolicy: "allowlist",
      groups: {
        "*": { allow: true, requireMention: true },
        "Work Chat": { allow: true, requireMention: false },
      },
    },
  },
}
```

## 多账号

账号映射到 OpenClaw 状态中的 `zalouser` 配置文件。示例：

```json
{
  channels: {
    zalouser: {
      enabled: true,
      defaultAccount: "default",
      accounts: {
        work: { enabled: true, profile: "work" },
      },
    },
  },
}
```

## 输入状态、反应和送达确认

-   OpenClaw 在发送回复前会发送一个输入状态事件（尽力而为）。
-   频道操作中支持 `zalouser` 的消息反应操作 `react`。
    -   使用 `remove: true` 从消息中移除特定的反应表情。
    -   反应语义：[反应](../tools/reactions.md)
-   对于包含事件元数据的入站消息，OpenClaw 会发送已送达 + 已读确认（尽力而为）。

## 故障排除

**登录不持久：**

-   `openclaw channels status --probe`
-   重新登录：`openclaw channels logout --channel zalouser && openclaw channels login --channel zalouser`

**允许列表/群组名称未解析：**

-   在 `allowFrom`/`groups` 中使用数字 ID，或确切的联系人/群组名称。

**从旧的基于 CLI 的设置升级：**

-   移除任何旧的关于外部 `zca` 进程的假设。
-   该频道现在完全在 OpenClaw 内部运行，无需外部 CLI 二进制文件。

[Zalo](./zalo.md)[配对](./pairing.md)