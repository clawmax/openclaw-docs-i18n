

  环境与调试

  
# Node + tsx 崩溃

本页介绍在运行 OpenClaw 网关或 CLI 时（例如 `node --import tsx` 或使用 tsx 的脚本）排查 **Node.js 与 tsx** 崩溃的方法。

## 常见原因

- **Node 版本**：OpenClaw 需要 Node 22+。更旧的 Node 可能导致运行时或原生模块崩溃。
- **tsx / 加载器**：启动或加载 TypeScript 时崩溃可能与 tsx 版本、`--import` 顺序或冲突的加载器有关。
- **内存 / 原生**：大负载或原生插件可能引发 OOM 或段错误；版本与环境说明见 [Node.js](../install/node.md)。

## 快速检查

1. **Node 版本**：运行 `node -v`（应为 v22 或更高）。
2. **不用 tsx 复现**：尝试用纯 `node` 与预编译 JS 跑同一流程，判断崩溃是否与 tsx 相关。
3. **诊断**：使用 [诊断标志](../diagnostics/flags.md)（如 `OPENCLAW_DEBUG_*`）在崩溃前获取更多日志。

## 相关链接

- [调试](../help/debugging.md) — 运行时覆盖、网关 watch、开发配置
- [脚本](../help/scripts.md) — 辅助脚本与约定
- [Node.js](../install/node.md) — Node 版本与 PATH
- [诊断标志](../diagnostics/flags.md) — 调试与跟踪标志
