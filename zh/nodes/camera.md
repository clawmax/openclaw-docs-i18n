title: "OpenClaw 中 iOS、Android、macOS 节点的摄像头捕获"
description: "学习如何在 OpenClaw 中使用摄像头节点捕获照片和视频。配置设置、使用 CLI 命令，并将摄像头捕获集成到智能体工作流中。"
keywords: ["摄像头捕获", "openclaw 节点", "ios 摄像头", "android 摄像头", "macos 摄像头", "node.invoke", "cli 助手", "媒体捕获"]
---

  媒体与设备

  
# 摄像头捕获

OpenClaw 支持智能体工作流的**摄像头捕获**功能：

-   **iOS 节点**（通过网关配对）：通过 `node.invoke` 捕获**照片**（`jpg`）或**短视频片段**（`mp4`，可选音频）。
-   **Android 节点**（通过网关配对）：通过 `node.invoke` 捕获**照片**（`jpg`）或**短视频片段**（`mp4`，可选音频）。
-   **macOS 应用**（通过网关的节点）：通过 `node.invoke` 捕获**照片**（`jpg`）或**短视频片段**（`mp4`，可选音频）。

所有摄像头访问均受**用户控制的设置**管理。

## iOS 节点

### 用户设置（默认开启）

-   iOS 设置标签页 → **摄像头** → **允许摄像头** (`camera.enabled`)
    -   默认值：**开启**（缺少键值视为开启）。
    -   关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`。

### 命令（通过网关 node.invoke）

-   `camera.list`
    -   响应负载：
        -   `devices`: `{ id, name, position, deviceType }` 的数组
-   `camera.snap`
    -   参数：
        -   `facing`: `front|back`（默认：`front`）
        -   `maxWidth`: 数字（可选；iOS 节点上默认为 `1600`）
        -   `quality`: `0..1`（可选；默认 `0.9`）
        -   `format`: 目前为 `jpg`
        -   `delayMs`: 数字（可选；默认 `0`）
        -   `deviceId`: 字符串（可选；来自 `camera.list`）
    -   响应负载：
        -   `format: "jpg"`
        -   `base64: "<...>"`
        -   `width`, `height`
    -   负载保护：照片会被重新压缩，以保持 base64 负载小于 5 MB。
-   `camera.clip`
    -   参数：
        -   `facing`: `front|back`（默认：`front`）
        -   `durationMs`: 数字（默认 `3000`，上限为 `60000`）
        -   `includeAudio`: 布尔值（默认 `true`）
        -   `format`: 目前为 `mp4`
        -   `deviceId`: 字符串（可选；来自 `camera.list`）
    -   响应负载：
        -   `format: "mp4"`
        -   `base64: "<...>"`
        -   `durationMs`
        -   `hasAudio`

### 前台要求

与 `canvas.*` 类似，iOS 节点只允许在**前台**执行 `camera.*` 命令。后台调用将返回 `NODE_BACKGROUND_UNAVAILABLE`。

### CLI 助手（临时文件 + MEDIA）

获取附件的最简单方法是使用 CLI 助手，它将解码后的媒体写入临时文件并打印 `MEDIA:<路径>`。示例：

```bash
openclaw nodes camera snap --node <id>               # 默认：前置 + 后置摄像头（输出 2 行 MEDIA）
openclaw nodes camera snap --node <id> --facing front
openclaw nodes camera clip --node <id> --duration 3000
openclaw nodes camera clip --node <id> --no-audio
```

注意：

-   `nodes camera snap` 默认使用**两个**摄像头朝向，以便为智能体提供两个视角。
-   输出文件是临时的（位于操作系统临时目录中），除非您构建自己的包装器。

## Android 节点

### Android 用户设置（默认开启）

-   Android 设置面板 → **摄像头** → **允许摄像头** (`camera.enabled`)
    -   默认值：**开启**（缺少键值视为开启）。
    -   关闭时：`camera.*` 命令返回 `CAMERA_DISABLED`。

### 权限

-   Android 需要运行时权限：
    -   `CAMERA` 权限用于 `camera.snap` 和 `camera.clip`。
    -   `RECORD_AUDIO` 权限用于 `camera.clip` 且 `includeAudio=true` 时。

如果缺少权限，应用将在可能时提示；如果被拒绝，`camera.*` 请求将失败并返回 `*_PERMISSION_REQUIRED` 错误。

### Android 前台要求

与 `canvas.*` 类似，Android 节点只允许在**前台**执行 `camera.*` 命令。后台调用将返回 `NODE_BACKGROUND_UNAVAILABLE`。

### Android 命令（通过网关 node.invoke）

-   `camera.list`
    -   响应负载：
        -   `devices`: `{ id, name, position, deviceType }` 的数组

### 负载保护

照片会被重新压缩，以保持 base64 负载小于 5 MB。

## macOS 应用

### 用户设置（默认关闭）

macOS 伴侣应用提供了一个复选框：

-   **设置 → 通用 → 允许摄像头** (`openclaw.cameraEnabled`)
    -   默认值：**关闭**
    -   关闭时：摄像头请求返回“用户已禁用摄像头”。

### CLI 助手（节点调用）

使用主 `openclaw` CLI 在 macOS 节点上调用摄像头命令。示例：

```bash
openclaw nodes camera list --node <id>            # 列出摄像头 id
openclaw nodes camera snap --node <id>            # 打印 MEDIA:<路径>
openclaw nodes camera snap --node <id> --max-width 1280
openclaw nodes camera snap --node <id> --delay-ms 2000
openclaw nodes camera snap --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --duration 10s          # 打印 MEDIA:<路径>
openclaw nodes camera clip --node <id> --duration-ms 3000      # 打印 MEDIA:<路径>（旧标志）
openclaw nodes camera clip --node <id> --device-id <id>
openclaw nodes camera clip --node <id> --no-audio
```

注意：

-   `openclaw nodes camera snap` 默认使用 `maxWidth=1600`，除非被覆盖。
-   在 macOS 上，`camera.snap` 会在预热/曝光稳定后等待 `delayMs`（默认 2000 毫秒）再进行捕获。
-   照片负载会被重新压缩，以保持 base64 小于 5 MB。

## 安全性与实际限制

-   摄像头和麦克风访问会触发通常的操作系统权限提示（并且需要在 Info.plist 中声明使用字符串）。
-   视频片段有上限（目前为 `<= 60s`），以避免节点负载过大（base64 开销 + 消息限制）。

## macOS 屏幕视频（操作系统级别）

对于*屏幕*视频（非摄像头），请使用 macOS 伴侣应用：

```bash
openclaw nodes screen record --node <id> --duration 10s --fps 15   # 打印 MEDIA:<路径>
```

注意：

-   需要 macOS **屏幕录制**权限（TCC）。

[音频和语音笔记](./audio.md)[通话模式](./talk.md)