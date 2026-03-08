

  CLI 命令

  
# update

安全更新 OpenClaw 并在稳定版/测试版/开发版频道之间切换。如果你通过 **npm/pnpm** 安装（全局安装，无 git 元数据），更新将通过包管理器流程在[更新指南](../install/updating.md)中进行。

## 用法

```bash
openclaw update
openclaw update status
openclaw update wizard
openclaw update --channel beta
openclaw update --channel dev
openclaw update --tag beta
openclaw update --dry-run
openclaw update --no-restart
openclaw update --json
openclaw --update
```

## 选项

-   `--no-restart`: 成功更新后跳过重启网关服务。
-   `--channel <stable|beta|dev>`: 设置更新频道（git + npm；保存在配置中）。
-   `--tag <dist-tag|version>`: 仅针对此次更新，覆盖 npm 分发标签或版本。
-   `--dry-run`: 预览计划的更新操作（频道/标签/目标/重启流程），而不写入配置、安装、同步插件或重启。
-   `--json`: 打印机器可读的 `UpdateRunResult` JSON。
-   `--timeout `: 每步超时时间（默认为 1200 秒）。

注意：降级需要确认，因为旧版本可能会破坏配置。

## update status

显示活跃的更新频道 + git 标签/分支/SHA（针对源码检出），以及更新可用性。

```bash
openclaw update status
openclaw update status --json
openclaw update status --timeout 10
```

选项：

-   `--json`: 打印机器可读的状态 JSON。
-   `--timeout `: 检查超时时间（默认为 3 秒）。

## update wizard

交互式流程，用于选择更新频道并确认更新后是否重启网关（默认重启）。如果你在没有 git 检出的情况下选择 `dev`，它会提示创建一个。

## 功能说明

当你显式切换频道（`--channel ...`）时，OpenClaw 也会保持安装方法一致：

-   `dev` → 确保有一个 git 检出（默认：`~/openclaw`，可通过 `OPENCLAW_GIT_DIR` 覆盖），更新它，并从该检出安装全局 CLI。
-   `stable`/`beta` → 使用匹配的分发标签从 npm 安装。

网关核心自动更新器（通过配置启用时）复用相同的更新路径。

## Git 检出流程

频道：

-   `stable`: 检出最新的非测试版标签，然后构建 + 运行 doctor。
-   `beta`: 检出最新的 `-beta` 标签，然后构建 + 运行 doctor。
-   `dev`: 检出 `main` 分支，然后获取 + 变基。

高级流程：

1.  要求工作树干净（无未提交的更改）。
2.  切换到选定的频道（标签或分支）。
3.  获取上游更新（仅限 dev）。
4.  仅限 dev：在临时工作树中进行预检 lint + TypeScript 构建；如果最新提交失败，则回溯最多 10 个提交以找到最新的干净构建。
5.  变基到选定的提交（仅限 dev）。
6.  安装依赖（优先使用 pnpm；npm 作为备选）。
7.  构建 + 构建控制界面。
8.  运行 `openclaw doctor` 作为最终的“安全更新”检查。
9.  将插件同步到活跃频道（dev 使用捆绑的扩展；stable/beta 使用 npm）并更新通过 npm 安装的插件。

## \--update 简写

`openclaw --update` 重写为 `openclaw update`（适用于 shell 和启动脚本）。

## 另请参阅

-   `openclaw doctor`（对于 git 检出，会首先提示运行更新）
-   [开发频道](../install/development-channels.md)
-   [更新指南](../install/updating.md)
-   [CLI 参考](../cli.md)

[uninstall](./uninstall.md)[voicecall](./voicecall.md)