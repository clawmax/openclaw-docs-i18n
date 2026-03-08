

  内置工具

  
# Reactions

跨渠道共享的反应语义：

-   添加反应时，`emoji` 是必需的。
-   `emoji=""` 在支持的情况下移除机器人在该消息上的所有反应。
-   `remove: true` 在支持的情况下移除指定的表情符号（需要 `emoji`）。

渠道注意事项：

-   **Discord/Slack**：空的 `emoji` 会移除机器人在该消息上的所有反应；`remove: true` 仅移除该表情符号。
-   **Google Chat**：空的 `emoji` 会移除应用在该消息上的反应；`remove: true` 仅移除该表情符号。
-   **Telegram**：空的 `emoji` 会移除机器人的反应；`remove: true` 也会移除反应，但工具验证仍需要一个非空的 `emoji`。
-   **WhatsApp**：空的 `emoji` 会移除机器人反应；`remove: true` 映射到空的表情符号（仍需要 `emoji`）。
-   **Zalo Personal (`zalouser`)**：需要非空的 `emoji`；`remove: true` 移除该特定的表情符号反应。
-   **Signal**：当启用 `channels.signal.reactionNotifications` 时，传入的反应通知会触发系统事件。

[工具循环检测](./loop-detection.md)[思考层级](./thinking.md)

---