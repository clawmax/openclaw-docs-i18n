

  macOS 伴侣应用

  
# 语音悬浮层

目标读者：macOS 应用贡献者。目标：在唤醒词和按键通话重叠时，保持语音悬浮层的可预测性。

## 当前意图

-   如果悬浮层已因唤醒词而显示，此时用户按下热键，热键会话将*沿用*现有文本而非重置它。悬浮层在热键按住期间保持显示。用户释放时：如果有修剪后的文本则发送，否则关闭。
-   单独的唤醒词仍会在静默时自动发送；按键通话则在释放时立即发送。

## 已实现 (2025年12月9日)

-   悬浮层会话现在为每次捕获（唤醒词或按键通话）携带一个令牌。当令牌不匹配时，部分/最终/发送/关闭/音量更新会被丢弃，避免了过时的回调。
-   按键通话会将任何可见的悬浮层文本作为前缀沿用（因此当唤醒悬浮层显示时按下热键会保留文本并追加新的语音）。它会等待最多1.5秒以获取最终转录，否则回退到当前文本。
-   提示音/悬浮层日志以 `info` 级别在类别 `voicewake.overlay`、`voicewake.ptt` 和 `voicewake.chime` 中发出（会话开始、部分、最终、发送、关闭、提示音原因）。

## 后续步骤

1.  **VoiceSessionCoordinator (actor)**
    -   一次只拥有一个 `VoiceSession`。
    -   API（基于令牌）：`beginWakeCapture`、`beginPushToTalk`、`updatePartial`、`endCapture`、`cancel`、`applyCooldown`。
    -   丢弃携带过时令牌的回调（防止旧的识别器重新打开悬浮层）。
2.  **VoiceSession (模型)**
    -   字段：`token`、`source` (wakeWord|pushToTalk)、已提交/易变文本、提示音标志、计时器（自动发送、空闲）、`overlayMode` (display|editing|sending)、冷却截止时间。
3.  **悬浮层绑定**
    -   `VoiceSessionPublisher` (`ObservableObject`) 将活动会话镜像到 SwiftUI。
    -   `VoiceWakeOverlayView` 仅通过发布者渲染；它从不直接修改全局单例。
    -   悬浮层用户操作 (`sendNow`、`dismiss`、`edit`) 使用会话令牌回调协调器。
4.  **统一发送路径**
    -   在 `endCapture` 时：如果修剪后的文本为空 → 关闭；否则 `performSend(session:)`（播放一次发送提示音，转发，关闭）。
    -   按键通话：无延迟；唤醒词：自动发送有可选延迟。
    -   在按键通话结束后对唤醒运行时应用短暂冷却，以防止唤醒词立即重新触发。
5.  **日志记录**
    -   协调器在子系统 `ai.openclaw`、类别 `voicewake.overlay` 和 `voicewake.chime` 中发出 `.info` 级别日志。
    -   关键事件：`session_started`、`adopted_by_push_to_talk`、`partial`、`finalized`、`send`、`dismiss`、`cancel`、`cooldown`。

## 调试清单

-   在复现悬浮层卡住问题时流式传输日志：
    
    复制
    
    ```bash
    sudo log stream --predicate 'subsystem == "ai.openclaw" AND category CONTAINS "voicewake"' --level info --style compact
    ```
    
-   验证只有一个活动会话令牌；过时的回调应被协调器丢弃。
-   确保按键通话释放时始终使用活动令牌调用 `endCapture`；如果文本为空，预期会`dismiss`且无提示音或发送。

## 迁移步骤（建议）

1.  添加 `VoiceSessionCoordinator`、`VoiceSession` 和 `VoiceSessionPublisher`。
2.  重构 `VoiceWakeRuntime`，使其创建/更新/结束会话，而不是直接操作 `VoiceWakeOverlayController`。
3.  重构 `VoicePushToTalk`，使其沿用现有会话并在释放时调用 `endCapture`；应用运行时冷却。
4.  将 `VoiceWakeOverlayController` 连接到发布者；移除来自 runtime/PTT 的直接调用。
5.  为会话沿用、冷却和空文本关闭添加集成测试。

[语音唤醒](./voicewake.md)[网页聊天](./webchat.md)

---