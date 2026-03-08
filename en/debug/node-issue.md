

  Environment and debugging

  
# Node + tsx Crash

This page covers troubleshooting **Node.js and tsx** crashes when running the OpenClaw gateway or CLI (e.g. `node --import tsx`, or scripts that use tsx).

## Common causes

- **Node version**: OpenClaw expects Node 22+. Older Node can cause runtime or native-module crashes.
- **tsx / loader**: Crashes during startup or when loading TypeScript can be due to tsx version, `--import` order, or conflicting loaders.
- **Memory / native**: Large workloads or native addons can trigger OOM or segfaults; see [Node.js](../install/node.md) for version and env guidance.

## Quick checks

1. **Node version**: Run `node -v` (expect v22 or higher).
2. **Reproduce without tsx**: Try running the same flow with plain `node` and precompiled JS to see if the crash is tsx-specific.
3. **Diagnostics**: Use [Diagnostics flags](../diagnostics/flags.md) (e.g. `OPENCLAW_DEBUG_*`) to get more logs before a crash.

## Related

- [Debugging](../help/debugging.md) — runtime overrides, gateway watch, dev profile
- [Scripts](../help/scripts.md) — helper scripts and conventions
- [Node.js](../install/node.md) — Node version and PATH
- [Diagnostics flags](../diagnostics/flags.md) — debug and trace flags
