

  Node runtime

  
# Node.js

OpenClaw requires **Node 22 or newer**. The [installer script](../install.md#install-methods) will detect and install Node automatically — this page is for when you want to set up Node yourself and make sure everything is wired up correctly (versions, PATH, global installs).

## Check your version

```bash
node -v
```

If this prints `v22.x.x` or higher, you’re good. If Node isn’t installed or the version is too old, pick an install method below.

## Install Node

 

Version managers let you switch between Node versions easily. Popular options:

-   [**fnm**](https://github.com/Schniz/fnm) — fast, cross-platform
-   [**nvm**](https://github.com/nvm-sh/nvm) — widely used on macOS/Linux
-   [**mise**](https://mise.jdx.dev/) — polyglot (Node, Python, Ruby, etc.)

Example with fnm:

```bash
fnm install 22
fnm use 22
```

Make sure your version manager is initialized in your shell startup file (`~/.zshrc` or `~/.bashrc`). If it isn’t, `openclaw` may not be found in new terminal sessions because the PATH won’t include Node’s bin directory.

## Troubleshooting

### openclaw: command not found

This almost always means npm’s global bin directory isn’t on your PATH. 

### Step 1: Find your global npm prefix

```bash
npm prefix -g
```

### Step 2: Check if it's on your PATH

```bash
echo "$PATH"
```

Look for `<npm-prefix>/bin` (macOS/Linux) or `<npm-prefix>` (Windows) in the output.

### Step 3: Add it to your shell startup file

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

Add the output of `npm prefix -g` to your system PATH via Settings → System → Environment Variables.

### Permission errors on npm install -g (Linux)

If you see `EACCES` errors, switch npm’s global prefix to a user-writable directory:

```bash
mkdir -p "$HOME/.npm-global"
npm config set prefix "$HOME/.npm-global"
export PATH="$HOME/.npm-global/bin:$PATH"
```

Add the `export PATH=...` line to your `~/.bashrc` or `~/.zshrc` to make it permanent.

[Diagnostics Flags](../diagnostics/flags.md)[Session Management Deep Dive](../reference/session-management-compaction.md)
