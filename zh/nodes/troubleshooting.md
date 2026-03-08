

  媒体与设备

  
# 节点故障排除

当节点状态可见但节点工具失败时，请使用此页面。

## 命令阶梯

```bash
openclaw status
openclaw gateway status
openclaw logs --follow
openclaw doctor
openclaw channels status --probe
```

然后运行节点特定检查：

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
```

健康信号：

-   节点已连接并针对 `node` 角色完成配对。
-   `nodes describe` 包含您正在调用的能力。
-   执行审批显示预期的模式/允许列表。

## 前台要求

`canvas.*`、`camera.*` 和 `screen.*` 在 iOS/Android 节点上仅支持前台操作。快速检查和修复：

```bash
openclaw nodes describe --node <idOrNameOrIp>
openclaw nodes canvas snapshot --node <idOrNameOrIp>
openclaw logs --follow
```

如果看到 `NODE_BACKGROUND_UNAVAILABLE`，请将节点应用切换到前台并重试。

## 权限矩阵

| 能力 | iOS | Android | macOS 节点应用 | 典型失败代码 |
| --- | --- | --- | --- | --- |
| `camera.snap`, `camera.clip` | 相机（+ 麦克风用于剪辑音频） | 相机（+ 麦克风用于剪辑音频） | 相机（+ 麦克风用于剪辑音频） | `*_PERMISSION_REQUIRED` |
| `screen.record` | 屏幕录制（+ 麦克风可选） | 屏幕捕获提示（+ 麦克风可选） | 屏幕录制 | `*_PERMISSION_REQUIRED` |
| `location.get` | 使用时或始终（取决于模式） | 基于模式的前台/后台位置 | 位置权限 | `LOCATION_PERMISSION_REQUIRED` |
| `system.run` | 不适用（节点主机路径） | 不适用（节点主机路径） | 需要执行审批 | `SYSTEM_RUN_DENIED` |

## 配对与审批

这些是不同的关卡：

1.  **设备配对**：此节点能否连接到网关？
2.  **执行审批**：此节点能否运行特定的 shell 命令？

快速检查：

```bash
openclaw devices list
openclaw nodes status
openclaw approvals get --node <idOrNameOrIp>
openclaw approvals allowlist add --node <idOrNameOrIp> "/usr/bin/uname"
```

如果缺少配对，请先批准节点设备。如果配对正常但 `system.run` 失败，请修复执行审批/允许列表。

## 常见节点错误代码

-   `NODE_BACKGROUND_UNAVAILABLE` → 应用在后台运行；将其切换到前台。
-   `CAMERA_DISABLED` → 节点设置中相机开关已禁用。
-   `*_PERMISSION_REQUIRED` → 操作系统权限缺失/被拒绝。
-   `LOCATION_DISABLED` → 位置模式已关闭。
-   `LOCATION_PERMISSION_REQUIRED` → 请求的位置模式未被授予。
-   `LOCATION_BACKGROUND_UNAVAILABLE` → 应用在后台运行，但仅拥有“使用时”权限。
-   `SYSTEM_RUN_DENIED: approval required` → 执行请求需要明确批准。
-   `SYSTEM_RUN_DENIED: allowlist miss` → 命令被允许列表模式阻止。在 Windows 节点主机上，像 `cmd.exe /c ...` 这样的 shell 包装形式在允许列表模式下会被视为允许列表未命中，除非通过询问流程批准。

## 快速恢复循环

```bash
openclaw nodes status
openclaw nodes describe --node <idOrNameOrIp>
openclaw approvals get --node <idOrNameOrIp>
openclaw logs --follow
```

如果仍然卡住：

-   重新批准设备配对。
-   重新打开节点应用（前台）。
-   重新授予操作系统权限。
-   重新创建/调整执行审批策略。

相关：

-   [/nodes/index](./index.md)
-   [/nodes/camera](./camera.md)
-   [/nodes/location-command](./location-command.md)
-   [/tools/exec-approvals](../tools/exec-approvals.md)
-   [/gateway/pairing](../gateway/pairing.md)

[节点](../nodes.md)[媒体理解](./media-understanding.md)