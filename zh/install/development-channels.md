

  高级

  
# 开发频道

最后更新：2026-01-21 OpenClaw 提供三个更新频道：

-   **稳定版**：npm dist-tag `latest`。
-   **测试版**：npm dist-tag `beta`（正在测试的构建）。
-   **开发版**：`main` 分支的最新提交（git）。npm dist-tag：`dev`（发布时）。

我们将构建发布到**测试版**，进行测试，然后**将经过审查的构建提升到 `latest`**，而无需更改版本号——dist-tags 是 npm 安装的真相源。

## 切换频道

Git checkout：

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

-   `stable`/`beta` 检出最新的匹配标签（通常是同一个标签）。
-   `dev` 切换到 `main` 分支并基于上游进行变基。

npm/pnpm 全局安装：

```bash
openclaw update --channel stable
openclaw update --channel beta
openclaw update --channel dev
```

这会通过相应的 npm dist-tag（`latest`、`beta`、`dev`）进行更新。当你**显式**使用 `--channel` 切换频道时，OpenClaw 还会对齐安装方法：

-   `dev` 确保有一个 git 检出（默认 `~/openclaw`，可通过 `OPENCLAW_GIT_DIR` 覆盖），更新它，并从该检出安装全局 CLI。
-   `stable`/`beta` 使用匹配的 dist-tag 从 npm 安装。

提示：如果你想同时拥有稳定版和开发版，请保留两个克隆，并将你的网关指向稳定版克隆。

## 插件与频道

当你使用 `openclaw update` 切换频道时，OpenClaw 还会同步插件源：

-   `dev` 优先使用 git 检出中的捆绑插件。
-   `stable` 和 `beta` 恢复通过 npm 安装的插件包。

## 标签最佳实践

-   为你希望 git 检出定位到的发布打标签（稳定版用 `vYYYY.M.D`，测试版用 `vYYYY.M.D-beta.N`）。
-   `vYYYY.M.D.beta.N` 为了兼容性也被识别，但更推荐使用 `-beta.N`。
-   旧的 `vYYYY.M.D-` 标签仍被识别为稳定版（非测试版）。
-   保持标签不可变：切勿移动或重用标签。
-   npm dist-tags 仍然是 npm 安装的真相源：
    -   `latest` → 稳定版
    -   `beta` → 候选构建
    -   `dev` → main 分支快照（可选）

## macOS 应用可用性

测试版和开发版构建**可能不**包含 macOS 应用发布。这没关系：

-   git 标签和 npm dist-tag 仍然可以发布。
-   在发布说明或更新日志中注明“此测试版无 macOS 构建”。

[在 Northflank 上部署](./northflank.md)

---