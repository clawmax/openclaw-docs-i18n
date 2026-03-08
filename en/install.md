

  Install overview

  
# Install

Already followed [Getting Started](./start/getting-started.md)? You’re all set — this page is for alternative install methods, platform-specific instructions, and maintenance.

## System requirements

-   **[Node 22+](./install/node.md)** (the [installer script](#install-methods) will install it if missing)
-   macOS, Linux, or Windows
-   `pnpm` only if you build from source

> **ℹ️** On Windows, we strongly recommend running OpenClaw under [WSL2](https://learn.microsoft.com/en-us/windows/wsl/install).

## Install methods

> **💡** The **installer script** is the recommended way to install OpenClaw. It handles Node detection, installation, and onboarding in one step.

 

> **⚠️** For VPS/cloud hosts, avoid third-party “1-click” marketplace images when possible. Prefer a clean base OS image (for example Ubuntu LTS), then install OpenClaw yourself with the installer script.

 

Downloads the CLI, installs it globally via npm, and launches the onboarding wizard.

That’s it — the script handles Node detection, installation, and onboarding.To skip onboarding and just install the binary:

For all flags, env vars, and CI/automation options, see [Installer internals](./install/installer.md).

If you already have Node 22+ and prefer to manage the install yourself:

For contributors or anyone who wants to run from a local checkout.

1

[

](#)

Clone and build

Clone the [OpenClaw repo](https://github.com/openclaw/openclaw) and build:

```bash
git clone https://github.com/openclaw/openclaw.git
cd openclaw
pnpm install
pnpm ui:build
pnpm build
```

2

[

](#)

Link the CLI

Make the `openclaw` command available globally:

```bash
pnpm link --global
```

Alternatively, skip the link and run commands via `pnpm openclaw ...` from inside the repo.

3

[

](#)

Run onboarding

```bash
openclaw onboard --install-daemon
```

For deeper development workflows, see [Setup](./start/setup.md).

## Other install methods

## After install

Verify everything is working:

```bash
openclaw doctor         # check for config issues
openclaw status         # gateway status
openclaw dashboard      # open the browser UI
```

If you need custom runtime paths, use:

-   `OPENCLAW_HOME` for home-directory based internal paths
-   `OPENCLAW_STATE_DIR` for mutable state location
-   `OPENCLAW_CONFIG_PATH` for config file location

See [Environment vars](./help/environment.md) for precedence and full details.

## Troubleshooting: openclaw not found

Quick diagnosis:

```bash
node -v
npm -v
npm prefix -g
echo "$PATH"
```

If `$(npm prefix -g)/bin` (macOS/Linux) or `$(npm prefix -g)` (Windows) is **not** in your `$PATH`, your shell can’t find global npm binaries (including `openclaw`).Fix — add it to your shell startup file (`~/.zshrc` or `~/.bashrc`):

```bash
export PATH="$(npm prefix -g)/bin:$PATH"
```

On Windows, add the output of `npm prefix -g` to your PATH.Then open a new terminal (or `rehash` in zsh / `hash -r` in bash).

## Update / uninstall

[Installer Internals](./install/installer.md)
