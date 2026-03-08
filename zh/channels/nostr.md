

  消息平台

  
# Nostr

**状态：** 可选插件（默认禁用）。Nostr 是一个用于社交网络的去中心化协议。此频道使 OpenClaw 能够通过 NIP-04 接收和响应加密直接消息（DM）。

## 安装（按需）

### 引导设置（推荐）

-   引导向导 (`openclaw onboard`) 和 `openclaw channels add` 会列出可选的频道插件。
-   选择 Nostr 会提示您按需安装插件。

安装默认项：

-   **开发频道 + git 检出可用：** 使用本地插件路径。
-   **稳定版/测试版：** 从 npm 下载。

您始终可以在提示中覆盖选择。

### 手动安装

```bash
openclaw plugins install @openclaw/nostr
```

使用本地检出（开发工作流）：

```bash
openclaw plugins install --link <path-to-openclaw>/extensions/nostr
```

安装或启用插件后，请重启网关。

## 快速设置

1.  生成 Nostr 密钥对（如果需要）：

```bash
# 使用 nak
nak key generate
```

2.  添加到配置：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}"
    }
  }
}
```

3.  导出密钥：

```bash
export NOSTR_PRIVATE_KEY="nsec1..."
```

4.  重启网关。

## 配置参考

| 键 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| `privateKey` | 字符串 | 必需 | 私钥，格式为 `nsec` 或十六进制 |
| `relays` | 字符串\[\] | `['wss://relay.damus.io', 'wss://nos.lol']` | 中继 URL（WebSocket） |
| `dmPolicy` | 字符串 | `pairing` | DM 访问策略 |
| `allowFrom` | 字符串\[\] | `[]` | 允许的发送者公钥 |
| `enabled` | 布尔值 | `true` | 启用/禁用频道 |
| `name` | 字符串 | \- | 显示名称 |
| `profile` | 对象 | \- | NIP-01 个人资料元数据 |

## 个人资料元数据

个人资料数据作为 NIP-01 `kind:0` 事件发布。您可以从控制界面（频道 -> Nostr -> 个人资料）管理它，或直接在配置中设置。示例：

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "profile": {
        "name": "openclaw",
        "displayName": "OpenClaw",
        "about": "个人助理 DM 机器人",
        "picture": "https://example.com/avatar.png",
        "banner": "https://example.com/banner.png",
        "website": "https://example.com",
        "nip05": "openclaw@example.com",
        "lud16": "openclaw@example.com"
      }
    }
  }
}
```

注意：

-   个人资料 URL 必须使用 `https://`。
-   从中继导入会合并字段并保留本地覆盖项。

## 访问控制

### DM 策略

-   **pairing**（默认）：未知发送者会收到一个配对码。
-   **allowlist**：只有 `allowFrom` 中的公钥可以发送 DM。
-   **open**：公开接收 DM（需要 `allowFrom: ["*"]`）。
-   **disabled**：忽略传入的 DM。

### 允许列表示例

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "dmPolicy": "allowlist",
      "allowFrom": ["npub1abc...", "npub1xyz..."]
    }
  }
}
```

## 密钥格式

接受的格式：

-   **私钥：** `nsec...` 或 64 字符十六进制
-   **公钥 (`allowFrom`):** `npub...` 或十六进制

## 中继

默认值：`relay.damus.io` 和 `nos.lol`。

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["wss://relay.damus.io", "wss://relay.primal.net", "wss://nostr.wine"]
    }
  }
}
```

提示：

-   使用 2-3 个中继以实现冗余。
-   避免使用太多中继（延迟、重复）。
-   付费中继可以提高可靠性。
-   本地中继适用于测试（`ws://localhost:7777`）。

## 协议支持

| NIP | 状态 | 描述 |
| --- | --- | --- |
| NIP-01 | 支持 | 基本事件格式 + 个人资料元数据 |
| NIP-04 | 支持 | 加密 DM (`kind:4`) |
| NIP-17 | 计划中 | 礼盒包装的 DM |
| NIP-44 | 计划中 | 版本化加密 |

## 测试

### 本地中继

```bash
# 启动 strfry
docker run -p 7777:7777 ghcr.io/hoytech/strfry
```

```json
{
  "channels": {
    "nostr": {
      "privateKey": "${NOSTR_PRIVATE_KEY}",
      "relays": ["ws://localhost:7777"]
    }
  }
}
```

### 手动测试

1.  从日志中记下机器人的公钥（npub）。
2.  打开一个 Nostr 客户端（Damus、Amethyst 等）。
3.  向该机器人公钥发送 DM。
4.  验证响应。

## 故障排除

### 未收到消息

-   验证私钥是否有效。
-   确保中继 URL 可达，并使用 `wss://`（本地使用 `ws://`）。
-   确认 `enabled` 不是 `false`。
-   检查网关日志中是否有中继连接错误。

### 未发送响应

-   检查中继是否接受写入。
-   验证出站连接性。
-   注意中继速率限制。

### 重复响应

-   使用多个中继时是预期行为。
-   消息通过事件 ID 去重；只有第一次投递会触发响应。

## 安全

-   切勿提交私钥。
-   对密钥使用环境变量。
-   对于生产环境机器人，考虑使用 `allowlist`。

## 限制（MVP）

-   仅支持直接消息（无群聊）。
-   无媒体附件。
-   仅支持 NIP-04（NIP-17 礼盒包装计划中）。

[Nextcloud Talk](./nextcloud-talk.md)[Signal](./signal.md)