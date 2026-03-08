

  媒体与设备

  
# 位置命令

## 摘要

-   `location.get` 是一个节点命令（通过 `node.invoke` 调用）。
-   默认关闭。
-   设置使用选择器：关闭 / 使用时 / 始终。
-   独立开关：精确位置。

## 为什么使用选择器（而非简单开关）

操作系统权限是多级的。我们可以在应用内展示选择器，但实际授权仍由操作系统决定。

-   **iOS/macOS**：用户可以在系统提示/设置中选择**使用时**或**始终**。应用可以请求升级，但操作系统可能要求前往设置。
-   **Android**：后台位置是单独的权限；在 Android 10+ 上，通常需要前往设置流程。
-   精确位置是单独的授权（iOS 14+ 的“精确”，Android 的“精确”与“粗略”）。

UI 中的选择器驱动我们请求的模式；实际授权存在于操作系统设置中。

## 设置模型

每个节点设备：

-   `location.enabledMode`: `off | whileUsing | always`
-   `location.preciseEnabled`: 布尔值

UI 行为：

-   选择 `whileUsing` 会请求前台权限。
-   选择 `always` 会首先确保 `whileUsing` 权限，然后请求后台权限（或在需要时将用户引导至设置）。
-   如果操作系统拒绝了请求的级别，则回退到已授予的最高级别并显示状态。

## 权限映射 (node.permissions)

可选。macOS 节点通过权限映射报告 `location`；iOS/Android 可能省略此项。

## 命令：location.get

通过 `node.invoke` 调用。参数（建议）：

```json
{
  "timeoutMs": 10000,
  "maxAgeMs": 15000,
  "desiredAccuracy": "coarse|balanced|precise"
}
```

响应负载：

```json
{
  "lat": 48.20849,
  "lon": 16.37208,
  "accuracyMeters": 12.5,
  "altitudeMeters": 182.0,
  "speedMps": 0.0,
  "headingDeg": 270.0,
  "timestamp": "2026-01-03T12:34:56.000Z",
  "isPrecise": true,
  "source": "gps|wifi|cell|unknown"
}
```

错误（稳定代码）：

-   `LOCATION_DISABLED`：选择器已关闭。
-   `LOCATION_PERMISSION_REQUIRED`：请求的模式缺少权限。
-   `LOCATION_BACKGROUND_UNAVAILABLE`：应用处于后台但仅允许“使用时”模式。
-   `LOCATION_TIMEOUT`：超时未获取定位。
-   `LOCATION_UNAVAILABLE`：系统故障 / 无可用定位源。

## 后台行为（未来）

目标：模型可以在节点处于后台时请求位置，但仅在以下条件下：

-   用户选择了**始终**。
-   操作系统授予后台位置权限。
-   应用被允许为位置服务在后台运行（iOS 后台模式 / Android 前台服务或特殊许可）。

推送触发流程（未来）：

1.  网关向节点发送推送（静默推送或 FCM 数据消息）。
2.  节点短暂唤醒并向设备请求位置。
3.  节点将负载转发给网关。

注意事项：

-   **iOS**：需要“始终”权限 + 后台位置模式。静默推送可能被节流；预计会出现间歇性失败。
-   **Android**：后台位置可能需要前台服务；否则，预计会被拒绝。

## 模型/工具集成

-   工具界面：`nodes` 工具添加 `location_get` 操作（需要节点）。
-   CLI：`openclaw nodes location get --node `。
-   代理指南：仅在用户启用了位置功能并了解其范围时调用。

## UX 文案（建议）

-   关闭：“位置共享已禁用。”
-   使用时：“仅在 OpenClaw 打开时。”
-   始终：“允许后台位置。需要系统权限。”
-   精确：“使用精确 GPS 位置。关闭此开关以共享大致位置。”

[语音唤醒](./voicewake.md)[文本转语音](../tts.md)

---