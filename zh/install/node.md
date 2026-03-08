

  Node 运行时

  
# Node.js

OpenClaw 需要 **Node 22 或更新版本**。[安装脚本](../install.md#install-methods) 会自动检测并安装 Node —— 本页面适用于您希望自行设置 Node 并确保所有配置正确（版本、PATH、全局安装）的情况。

## 检查您的版本

```bash
node -v
```

如果此命令打印 `v22.x.x` 或更高版本，则说明一切正常。如果未安装 Node 或版本过旧，请选择下面的安装方法。

## 安装 Node

 

版本管理器让您可以轻松切换 Node 版本。流行的选项包括：

-   [**fnm**](https://github.com/Schniz/fnm) —— 快速，跨平台
-   [**nvm**](https://github.com/nvm-sh/nvm) —— 在 macOS/Linux 上广泛使用
-   [**mise**](https://mise.jdx.dev/) —— 多语言支持（Node、Python、Ruby 等）

使用 fnm 的示例：

```bash
fnm install 22
fnm use 22
```

请确保您的版本管理器已在您的 shell 启动文件（`~/.zshrc` 或 `~/.bashrc`）中初始化。如果未初始化，在新的终端会话中可能找不到 `openclaw` 命令，因为 PATH 将不包含 Node 的 bin 目录。

## 故障排除

### openclaw: command not found

这几乎总是意味着 npm 的全局 bin 目录不在您的 PATH 中。

### 步骤 1：查找您的全局 npm 前缀

```bash
npm prefix -g
```

### 步骤 2：检查它是否在您的 PATH 中

```bash
echo "$PATH"
```

在输出中查找 `<npm-prefix>/bin` (macOS/Linux) 或 `<npm-prefix>` (Windows)。

### 步骤 3：将其添加到您的 shell 启动文件

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

通过“设置” → “系统” → “环境变量”，将 `npm prefix -g` 的输出添加到您的系统 PATH 中。

### npm install -g 权限错误 (Linux)

如果您看到 `EACCES` 错误，请将 npm 的全局前缀切换为用户可写的目录：

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

将 `export PATH=...` 这行添加到您的 `~/.bashrc` 或 `~/.zshrc` 中以使其永久生效。

[诊断标志](../diagnostics/flags.md)[会话管理深度解析](../reference/session-management-compaction.md)

---