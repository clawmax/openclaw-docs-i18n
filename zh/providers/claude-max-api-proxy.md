

  提供商

  
# Claude Max API 代理

**claude-max-api-proxy** 是一个社区工具，可将您的 Claude Max/Pro 订阅作为 OpenAI 兼容的 API 端点暴露。这允许您在任何支持 OpenAI API 格式的工具中使用您的订阅。

> **⚠️** 此路径仅为技术兼容性。Anthropic 过去曾阻止过在 Claude Code 之外使用某些订阅。您必须自行决定是否使用它，并在依赖它之前核实 Anthropic 的当前条款。

## 为什么使用这个？

| 方法 | 成本 | 最适合 |
| --- | --- | --- |
| Anthropic API | 按 token 付费（Opus 约 15美元/百万输入，75美元/百万输出） | 生产应用，高用量 |
| Claude Max 订阅 | 200美元/月（固定） | 个人使用，开发，无限用量 |

如果您拥有 Claude Max 订阅，并希望将其用于 OpenAI 兼容的工具，此代理可能会降低某些工作流的成本。对于生产用途，API 密钥仍然是政策上更清晰的路径。

## 工作原理

```
您的应用 → claude-max-api-proxy → Claude Code CLI → Anthropic（通过订阅）
     (OpenAI 格式)              (转换格式)      (使用您的登录信息)
```

该代理：

1.  在 `http://localhost:3456/v1/chat/completions` 接受 OpenAI 格式的请求
2.  将其转换为 Claude Code CLI 命令
3.  以 OpenAI 格式返回响应（支持流式传输）

## 安装

```bash
# 需要 Node.js 20+ 和 Claude Code CLI
npm install -g claude-max-api-proxy

# 验证 Claude CLI 已认证
claude --version
```

## 使用方法

### 启动服务器

```
claude-max-api
# 服务器运行在 http://localhost:3456
```

### 测试

```bash
# 健康检查
curl http://localhost:3456/health

# 列出模型
curl http://localhost:3456/v1/models

# 聊天补全
curl http://localhost:3456/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{
    "model": "claude-opus-4",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'
```

### 与 OpenClaw 配合使用

您可以将 OpenClaw 指向该代理作为自定义的 OpenAI 兼容端点：

```json
{
  env: {
    OPENAI_API_KEY: "not-needed",
    OPENAI_BASE_URL: "http://localhost:3456/v1",
  },
  agents: {
    defaults: {
      model: { primary: "openai/claude-opus-4" },
    },
  },
}
```

## 可用模型

| 模型 ID | 映射到 |
| --- | --- |
| `claude-opus-4` | Claude Opus 4 |
| `claude-sonnet-4` | Claude Sonnet 4 |
| `claude-haiku-4` | Claude Haiku 4 |

## 在 macOS 上自动启动

创建一个 LaunchAgent 来自动运行代理：

```bash
cat > ~/Library/LaunchAgents/com.claude-max-api.plist << 'EOF'
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>com.claude-max-api</string>
  <key>RunAtLoad</key>
  <true/>
  <key>KeepAlive</key>
  <true/>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/bin/node</string>
    <string>/usr/local/lib/node_modules/claude-max-api-proxy/dist/server/standalone.js</string>
  </array>
  <key>EnvironmentVariables</key>
  <dict>
    <key>PATH</key>
    <string>/usr/local/bin:/opt/homebrew/bin:~/.local/bin:/usr/bin:/bin</string>
  </dict>
</dict>
</plist>
EOF

launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.claude-max-api.plist
```

## 链接

-   **npm:** [https://www.npmjs.com/package/claude-max-api-proxy](https://www.npmjs.com/package/claude-max-api-proxy)
-   **GitHub:** [https://github.com/atalovesyou/claude-max-api-proxy](https://github.com/atalovesyou/claude-max-api-proxy)
-   **问题反馈:** [https://github.com/atalovesyou/claude-max-api-proxy/issues](https://github.com/atalovesyou/claude-max-api-proxy/issues)

## 注意事项

-   这是一个**社区工具**，并非由 Anthropic 或 OpenClaw 官方支持
-   需要有效的 Claude Max/Pro 订阅，且 Claude Code CLI 已认证
-   代理在本地运行，不会将数据发送到任何第三方服务器
-   完全支持流式响应

## 另请参阅

-   [Anthropic 提供商](./anthropic.md) - 使用 Claude setup-token 或 API 密钥的原生 OpenClaw 集成
-   [OpenAI 提供商](./openai.md) - 用于 OpenAI/Codex 订阅

[Cloudflare AI 网关](./cloudflare-ai-gateway.md)[Deepgram](./deepgram.md)