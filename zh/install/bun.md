

  其他安装方法

  
# Bun (实验性)

目标：使用 **Bun**（可选，不推荐用于 WhatsApp/Telegram）运行此仓库，同时不偏离 pnpm 的工作流程。⚠️ **不推荐用于网关运行时**（WhatsApp/Telegram 存在 bug）。生产环境请使用 Node。

## 状态

-   Bun 是一个可选的本地运行时，用于直接运行 TypeScript（`bun run …`、`bun --watch …`）。
-   `pnpm` 是默认的构建工具，并保持完全支持（且被部分文档工具使用）。
-   Bun 无法使用 `pnpm-lock.yaml` 并会忽略它。

## 安装

默认：

```bash
bun install
```

注意：`bun.lock`/`bun.lockb` 已被 git 忽略，因此无论哪种方式都不会造成仓库混乱。如果你希望*不写入任何锁文件*：

```bash
bun install --no-save
```

## 构建 / 测试 (Bun)

```bash
bun run build
bun run vitest run
```

## Bun 生命周期脚本（默认被阻止）

除非明确信任（`bun pm untrusted` / `bun pm trust`），否则 Bun 可能会阻止依赖项的生命周期脚本。对于此仓库，通常被阻止的脚本并非必需：

-   `@whiskeysockets/baileys` 的 `preinstall`：检查 Node 主版本 >= 20（我们运行 Node 22+）。
-   `protobufjs` 的 `postinstall`：发出关于不兼容版本方案的警告（不产生构建产物）。

如果你遇到确实需要这些脚本的运行时问题，请明确信任它们：

```bash
bun pm trust @whiskeysockets/baileys protobufjs
```

## 注意事项

-   部分脚本仍然硬编码了 pnpm（例如 `docs:build`、`ui:*`、`protocol:check`）。目前请通过 pnpm 运行这些脚本。

[Ansible](./ansible.md)[更新](./updating.md)

---