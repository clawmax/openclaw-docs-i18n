

  Hosting and deployment

  
# VPS Hosting

This hub links to the supported VPS/hosting guides and explains how cloud deployments work at a high level.

## Pick a provider

-   **Railway** (one‑click + browser setup): [Railway](./install/railway.md)
-   **Northflank** (one‑click + browser setup): [Northflank](./install/northflank.md)
-   **Oracle Cloud (Always Free)**: [Oracle](./platforms/oracle.md) — $0/month (Always Free, ARM; capacity/signup can be finicky)
-   **Fly.io**: [Fly.io](./install/fly.md)
-   **Hetzner (Docker)**: [Hetzner](./install/hetzner.md)
-   **GCP (Compute Engine)**: [GCP](./install/gcp.md)
-   **exe.dev** (VM + HTTPS proxy): [exe.dev](./install/exe-dev.md)
-   **AWS (EC2/Lightsail/free tier)**: works well too. Video guide: [https://x.com/techfrenAJ/status/2014934471095812547](https://x.com/techfrenAJ/status/2014934471095812547)

## How cloud setups work

-   The **Gateway runs on the VPS** and owns state + workspace.
-   You connect from your laptop/phone via the **Control UI** or **Tailscale/SSH**.
-   Treat the VPS as the source of truth and **back up** the state + workspace.
-   Secure default: keep the Gateway on loopback and access it via SSH tunnel or Tailscale Serve. If you bind to `lan`/`tailnet`, require `gateway.auth.token` or `gateway.auth.password`.

Remote access: [Gateway remote](./gateway/remote.md)  
Platforms hub: [Platforms](./platforms.md)

## Shared company agent on a VPS

This is a valid setup when the users are in one trust boundary (for example one company team), and the agent is business-only.

-   Keep it on a dedicated runtime (VPS/VM/container + dedicated OS user/accounts).
-   Do not sign that runtime into personal Apple/Google accounts or personal browser/password-manager profiles.
-   If users are adversarial to each other, split by gateway/host/OS user.

Security model details: [Security](./gateway/security.md)

## Using nodes with a VPS

You can keep the Gateway in the cloud and pair **nodes** on your local devices (Mac/iOS/Android/headless). Nodes provide local screen/camera/canvas and `system.run` capabilities while the Gateway stays in the cloud. Docs: [Nodes](./nodes.md), [Nodes CLI](./cli/nodes.md)

## Startup tuning for small VMs and ARM hosts

If CLI commands feel slow on low-power VMs (or ARM hosts), enable Node’s module compile cache:

```
grep -q 'NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache' ~/.bashrc || cat >> ~/.bashrc <<'EOF'
export NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
mkdir -p /var/tmp/openclaw-compile-cache
export OPENCLAW_NO_RESPAWN=1
EOF
source ~/.bashrc
```

-   `NODE_COMPILE_CACHE` improves repeated command startup times.
-   `OPENCLAW_NO_RESPAWN=1` avoids extra startup overhead from a self-respawn path.
-   First command run warms cache; subsequent runs are faster.
-   For Raspberry Pi specifics, see [Raspberry Pi](./platforms/raspberry-pi.md).

### systemd tuning checklist (optional)

For VM hosts using `systemd`, consider:

-   Add service env for stable startup path:
    -   `OPENCLAW_NO_RESPAWN=1`
    -   `NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache`
-   Keep restart behavior explicit:
    -   `Restart=always`
    -   `RestartSec=2`
    -   `TimeoutStartSec=90`
-   Prefer SSD-backed disks for state/cache paths to reduce random-I/O cold-start penalties.

Example:

```bash
sudo systemctl edit openclaw
```

```ini
[Service]
Environment=OPENCLAW_NO_RESPAWN=1
Environment=NODE_COMPILE_CACHE=/var/tmp/openclaw-compile-cache
Restart=always
RestartSec=2
TimeoutStartSec=90
```

How `Restart=` policies help automated recovery: [systemd can automate service recovery](https://www.redhat.com/en/blog/systemd-automate-recovery).

[Uninstall](./install/uninstall.md)[Fly.io](./install/fly.md)
