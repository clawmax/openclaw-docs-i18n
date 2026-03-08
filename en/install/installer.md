

  Install overview

  
# Installer Internals

OpenClaw ships three installer scripts, served from `openclaw.ai`.

| Script | Platform | What it does |
| --- | --- | --- |
| [`install.sh`](#installsh) | macOS / Linux / WSL | Installs Node if needed, installs OpenClaw via npm (default) or git, and can run onboarding. |
| [`install-cli.sh`](#install-clish) | macOS / Linux / WSL | Installs Node + OpenClaw into a local prefix (`~/.openclaw`). No root required. |
| [`install.ps1`](#installps1) | Windows (PowerShell) | Installs Node if needed, installs OpenClaw via npm (default) or git, and can run onboarding. |

## Quick commands

 

> **ℹ️** If install succeeds but `openclaw` is not found in a new terminal, see [Node.js troubleshooting](./node.md#troubleshooting).

* * *

## install.sh

> **💡** Recommended for most interactive installs on macOS/Linux/WSL.

### Flow (install.sh)

### Step 1: Detect OS

Supports macOS and Linux (including WSL). If macOS is detected, installs Homebrew if missing.

### Step 2: Ensure Node.js 22+

Checks Node version and installs Node 22 if needed (Homebrew on macOS, NodeSource setup scripts on Linux apt/dnf/yum).

### Step 3: Ensure Git

Installs Git if missing.

### Step 4: Install OpenClaw

-   `npm` method (default): global npm install
-   `git` method: clone/update repo, install deps with pnpm, build, then install wrapper at `~/.local/bin/openclaw`

### Step 5: Post-install tasks

-   Runs `openclaw doctor --non-interactive` on upgrades and git installs (best effort)
-   Attempts onboarding when appropriate (TTY available, onboarding not disabled, and bootstrap/config checks pass)
-   Defaults `SHARP_IGNORE_GLOBAL_LIBVIPS=1`

### Source checkout detection

If run inside an OpenClaw checkout (`package.json` + `pnpm-workspace.yaml`), the script offers:

-   use checkout (`git`), or
-   use global install (`npm`)

If no TTY is available and no install method is set, it defaults to `npm` and warns. The script exits with code `2` for invalid method selection or invalid `--install-method` values.

### Examples (install.sh)

 

| Flag | Description |
| --- | --- |
| `--install-method npm\|git` | Choose install method (default: `npm`). Alias: `--method` |
| `--npm` | Shortcut for npm method |
| `--git` | Shortcut for git method. Alias: `--github` |
| `--version <version\|dist-tag>` | npm version or dist-tag (default: `latest`) |
| `--beta` | Use beta dist-tag if available, else fallback to `latest` |
| `--git-dir ` | Checkout directory (default: `~/openclaw`). Alias: `--dir` |
| `--no-git-update` | Skip `git pull` for existing checkout |
| `--no-prompt` | Disable prompts |
| `--no-onboard` | Skip onboarding |
| `--onboard` | Enable onboarding |
| `--dry-run` | Print actions without applying changes |
| `--verbose` | Enable debug output (`set -x`, npm notice-level logs) |
| `--help` | Show usage (`-h`) |

| Variable | Description |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Install method |
| `OPENCLAW_VERSION=latest\|next\|` | npm version or dist-tag |
| `OPENCLAW_BETA=0\|1` | Use beta if available |
| `OPENCLAW_GIT_DIR=` | Checkout directory |
| `OPENCLAW_GIT_UPDATE=0\|1` | Toggle git updates |
| `OPENCLAW_NO_PROMPT=1` | Disable prompts |
| `OPENCLAW_NO_ONBOARD=1` | Skip onboarding |
| `OPENCLAW_DRY_RUN=1` | Dry run mode |
| `OPENCLAW_VERBOSE=1` | Debug mode |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm log level |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | Control sharp/libvips behavior (default: `1`) |

* * *

## install-cli.sh

> **ℹ️** Designed for environments where you want everything under a local prefix (default `~/.openclaw`) and no system Node dependency.

### Flow (install-cli.sh)

### Step 1: Install local Node runtime

Downloads Node tarball (default `22.22.0`) to `/tools/node-v` and verifies SHA-256.

### Step 2: Ensure Git

If Git is missing, attempts install via apt/dnf/yum on Linux or Homebrew on macOS.

### Step 3: Install OpenClaw under prefix

Installs with npm using `--prefix `, then writes wrapper to `/bin/openclaw`.

### Examples (install-cli.sh)

 

| Flag | Description |
| --- | --- |
| `--prefix ` | Install prefix (default: `~/.openclaw`) |
| `--version ` | OpenClaw version or dist-tag (default: `latest`) |
| `--node-version ` | Node version (default: `22.22.0`) |
| `--json` | Emit NDJSON events |
| `--onboard` | Run `openclaw onboard` after install |
| `--no-onboard` | Skip onboarding (default) |
| `--set-npm-prefix` | On Linux, force npm prefix to `~/.npm-global` if current prefix is not writable |
| `--help` | Show usage (`-h`) |

| Variable | Description |
| --- | --- |
| `OPENCLAW_PREFIX=` | Install prefix |
| `OPENCLAW_VERSION=` | OpenClaw version or dist-tag |
| `OPENCLAW_NODE_VERSION=` | Node version |
| `OPENCLAW_NO_ONBOARD=1` | Skip onboarding |
| `OPENCLAW_NPM_LOGLEVEL=error\|warn\|notice` | npm log level |
| `OPENCLAW_GIT_DIR=` | Legacy cleanup lookup path (used when removing old `Peekaboo` submodule checkout) |
| `SHARP_IGNORE_GLOBAL_LIBVIPS=0\|1` | Control sharp/libvips behavior (default: `1`) |

* * *

## install.ps1

### Flow (install.ps1)

### Step 1: Ensure PowerShell + Windows environment

Requires PowerShell 5+.

### Step 2: Ensure Node.js 22+

If missing, attempts install via winget, then Chocolatey, then Scoop.

### Step 3: Install OpenClaw

-   `npm` method (default): global npm install using selected `-Tag`
-   `git` method: clone/update repo, install/build with pnpm, and install wrapper at `%USERPROFILE%\.local\bin\openclaw.cmd`

### Step 4: Post-install tasks

Adds needed bin directory to user PATH when possible, then runs `openclaw doctor --non-interactive` on upgrades and git installs (best effort).

### Examples (install.ps1)

 

| Flag | Description |
| --- | --- |
| `-InstallMethod npm\|git` | Install method (default: `npm`) |
| `-Tag ` | npm dist-tag (default: `latest`) |
| `-GitDir ` | Checkout directory (default: `%USERPROFILE%\openclaw`) |
| `-NoOnboard` | Skip onboarding |
| `-NoGitUpdate` | Skip `git pull` |
| `-DryRun` | Print actions only |

| Variable | Description |
| --- | --- |
| `OPENCLAW_INSTALL_METHOD=git\|npm` | Install method |
| `OPENCLAW_GIT_DIR=` | Checkout directory |
| `OPENCLAW_NO_ONBOARD=1` | Skip onboarding |
| `OPENCLAW_GIT_UPDATE=0` | Disable git pull |
| `OPENCLAW_DRY_RUN=1` | Dry run mode |

 

> **ℹ️** If `-InstallMethod git` is used and Git is missing, the script exits and prints the Git for Windows link.

* * *

## CI and automation

Use non-interactive flags/env vars for predictable runs. 

* * *

## Troubleshooting

Git is required for `git` install method. For `npm` installs, Git is still checked/installed to avoid `spawn git ENOENT` failures when dependencies use git URLs.

Some Linux setups point npm global prefix to root-owned paths. `install.sh` can switch prefix to `~/.npm-global` and append PATH exports to shell rc files (when those files exist).

The scripts default `SHARP_IGNORE_GLOBAL_LIBVIPS=1` to avoid sharp building against system libvips. To override:

```
SHARP_IGNORE_GLOBAL_LIBVIPS=0 curl -fsSL --proto '=https' --tlsv1.2 https://openclaw.ai/install.sh | bash
```

Install Git for Windows, reopen PowerShell, rerun installer.

Run `npm config get prefix` and add that directory to your user PATH (no `\bin` suffix needed on Windows), then reopen PowerShell.

`install.ps1` does not currently expose a `-Verbose` switch. Use PowerShell tracing for script-level diagnostics:

```powershell
Set-PSDebug -Trace 1
& ([scriptblock]::Create((iwr -useb https://openclaw.ai/install.ps1))) -NoOnboard
Set-PSDebug -Trace 0
```

Usually a PATH issue. See [Node.js troubleshooting](./node.md#troubleshooting).

[Install](../install.md)[Docker](./docker.md)
