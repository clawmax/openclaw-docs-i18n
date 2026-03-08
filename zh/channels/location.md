

  配置

  
# 频道位置解析

OpenClaw 将聊天频道中分享的位置规范化为：

-   附加到入站正文的人类可读文本，以及
-   自动回复上下文负载中的结构化字段。

目前支持：

-   **Telegram**（位置图钉 + 地点 + 实时位置）
-   **WhatsApp**（位置消息 + 实时位置消息）
-   **Matrix**（带有 `geo_uri` 的 `m.location`）

## 文本格式

位置被渲染为不带括号的友好行：

-   图钉：
    -   `📍 48.858844, 2.294351 ±12m`
-   命名地点：
    -   `📍 埃菲尔铁塔 — 战神广场，巴黎 (48.858844, 2.294351 ±12m)`
-   实时分享：
    -   `🛰 实时位置：48.858844, 2.294351 ±12m`

如果频道包含标题/评论，则将其附加在下一行：

```
📍 48.858844, 2.294351 ±12m
在这里见面
```

## 上下文字段

当存在位置时，这些字段会被添加到 `ctx` 中：

-   `LocationLat`（数字）
-   `LocationLon`（数字）
-   `LocationAccuracy`（数字，单位米；可选）
-   `LocationName`（字符串；可选）
-   `LocationAddress`（字符串；可选）
-   `LocationSource`（`pin | place | live`）
-   `LocationIsLive`（布尔值）

## 频道说明

-   **Telegram**：地点映射到 `LocationName`/`LocationAddress`；实时位置使用 `live_period`。
-   **WhatsApp**：`locationMessage.comment` 和 `liveLocationMessage.caption` 作为标题行附加。
-   **Matrix**：`geo_uri` 被解析为图钉位置；忽略海拔高度，且 `LocationIsLive` 始终为 false。

[频道路由](./channel-routing.md)[频道故障排除](./troubleshooting.md)

---