

  媒体与设备

  
# 语音唤醒

OpenClaw将**唤醒词视为一个单一的全局列表**，由**网关**所有。

-   **没有针对每个节点的自定义唤醒词**。
-   **任何节点/应用界面都可以编辑**该列表；更改由网关持久化并广播给所有节点。
-   macOS和iOS保留本地的**语音唤醒启用/禁用**开关（本地用户体验和权限不同）。
-   Android目前保持语音唤醒关闭，并在语音标签页中使用手动麦克风流程。

## 存储（网关主机）

唤醒词存储在网关机器的以下位置：

-   `~/.openclaw/settings/voicewake.json`

结构：

```json
{ "triggers": ["openclaw", "claude", "computer"], "updatedAtMs": 1730000000000 }
```

## 协议

### 方法

-   `voicewake.get` → `{ triggers: string[] }`
-   `voicewake.set` 参数为 `{ triggers: string[] }` → `{ triggers: string[] }`

注意：

-   触发器会被规范化（修剪、丢弃空值）。空列表会回退到默认值。
-   为安全起见，会强制执行限制（数量/长度上限）。

### 事件

-   `voicewake.changed` 载荷 `{ triggers: string[] }`

接收者：

-   所有WebSocket客户端（macOS应用、WebChat等）
-   所有已连接的节点（iOS/Android），并且在节点连接时也会作为初始“当前状态”推送。

## 客户端行为

### macOS应用

-   使用全局列表来控制 `VoiceWakeRuntime` 的触发。
-   在语音唤醒设置中编辑“触发词”会调用 `voicewake.set`，然后依赖广播来保持其他客户端同步。

### iOS节点

-   使用全局列表进行 `VoiceWakeManager` 触发检测。
-   在设置中编辑唤醒词会调用 `voicewake.set`（通过网关WebSocket），并保持本地唤醒词检测的响应性。

### Android节点

-   目前Android运行时/设置中禁用了语音唤醒。
-   Android语音功能在语音标签页中使用手动麦克风捕获，而不是唤醒词触发。

[对话模式](./talk.md)[位置指令](./location-command.md)

---