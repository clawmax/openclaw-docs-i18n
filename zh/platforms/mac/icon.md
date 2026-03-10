

  macOS 伴侣应用

  
# 菜单栏图标

作者: steipete · 更新于: 2025-12-06 · 范围: macOS 应用 (`apps/macos`)

-   **空闲：** 正常图标动画（眨眼，偶尔摆动）。
-   **暂停：** 状态项使用 `appearsDisabled`；无动作。
-   **语音触发（大耳朵）：** 语音唤醒检测器在听到唤醒词时调用 `AppState.triggerVoiceEars(ttl: nil)`，在捕获话语期间保持 `earBoostActive=true`。耳朵放大（1.9 倍），为清晰度显示圆形耳洞，然后在 1 秒静默后通过 `stopVoiceEars()` 恢复。仅由应用内语音管道触发。
-   **工作中（智能体运行）：** `AppState.isWorking=true` 驱动"尾巴/腿快速摆动"微动：工作期间腿部摆动更快并伴有轻微偏移。目前围绕 WebChat 智能体（agent）运行进行切换；在连接其他长时间任务时添加相同的切换。

连接点

-   语音唤醒：运行时/测试器在触发时调用 `AppState.triggerVoiceEars(ttl: nil)`，并在 1 秒静默后调用 `stopVoiceEars()` 以匹配捕获窗口。
-   智能体活动：在工作时段前后设置 `AppStateStore.shared.setWorking(true/false)`（已在 WebChat 智能体调用中完成）。保持时段短暂并在 `defer` 块中重置，以避免动画卡住。

形状与尺寸

-   基础图标在 `CritterIconRenderer.makeIcon(blink:legWiggle:earWiggle:earScale:earHoles:)` 中绘制。
-   耳朵缩放默认 `1.0`；语音增强设置 `earScale=1.9` 并切换 `earHoles=true`，不改变整体框架（18×18 pt 模板图像渲染到 36×36 px Retina 后备存储中）。
-   快速摆动使用腿部摆动，幅度可达约 1.0，并伴有小的水平抖动；它是现有空闲摆动的叠加。

行为说明

-   没有用于耳朵/工作状态的外部 CLI/智能体切换；保持其仅由应用自身信号内部控制，以避免意外抖动。
-   保持 TTL 短暂（&lt;10 秒），以便在任务挂起时图标能快速返回基线状态。

[健康检查](./health.md)[macOS 日志记录](./logging.md)

---