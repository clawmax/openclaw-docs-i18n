

  RPC 和 API

  
# 设备型号数据库

macOS 配套应用通过将苹果型号标识符（例如 `iPad16,6`、`Mac16,6`）映射为人类可读的名称，在 **实例** 用户界面中显示友好的苹果设备型号名称。该映射以 JSON 形式提供，位于：

-   `apps/macos/Sources/OpenClaw/Resources/DeviceModels/`

## 数据源

我们目前从以下 MIT 许可的仓库获取映射数据：

-   `kyle-seongwoo-jun/apple-device-identifiers`

为了保持构建的确定性，JSON 文件被固定到特定的上游提交（记录在 `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md` 中）。

## 更新数据库

1.  选择你想要固定的上游提交（一个用于 iOS，一个用于 macOS）。
2.  更新 `apps/macos/Sources/OpenClaw/Resources/DeviceModels/NOTICE.md` 中的提交哈希值。
3.  重新下载 JSON 文件，固定到这些提交：

```
IOS_COMMIT="<ios-device-identifiers.json 的提交 sha>"
MAC_COMMIT="<mac-device-identifiers.json 的提交 sha>"

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${IOS_COMMIT}/ios-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/ios-device-identifiers.json

curl -fsSL "https://raw.githubusercontent.com/kyle-seongwoo-jun/apple-device-identifiers/${MAC_COMMIT}/mac-device-identifiers.json" \
  -o apps/macos/Sources/OpenClaw/Resources/DeviceModels/mac-device-identifiers.json
```

4.  确保 `apps/macos/Sources/OpenClaw/Resources/DeviceModels/LICENSE.apple-device-identifiers.txt` 仍然与上游匹配（如果上游许可证发生变化，请替换它）。
5.  验证 macOS 应用构建无误（无警告）：

```bash
swift build --package-path apps/macos
```

[RPC 适配器](./rpc.md)[默认 AGENTS.md](./AGENTS.default.md)

---