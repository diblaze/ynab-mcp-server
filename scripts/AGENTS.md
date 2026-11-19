<!-- Parent: ../AGENTS.md -->
# scripts/

## Purpose
Release and build tooling, two offline gates of `npm test` (the safety model and the scripted agent eval), and operator smoke checks that talk to a live or hosted MCP server.

## Key files
| File | Purpose |
|------|---------|
| `build-mcpb.mjs` | Builds `dist/mcp-server-for-ynab-<ver>.mcpb` (`build-mcpb.mjs:13`) |
| `check-dependencies.mjs` | Verifies `sdk`, `ynab`, `zod` resolve; installs nothing (`check-dependencies.mjs:3`) |
| `check-release-consistency.mjs` | Release consistency assertions, optional `--registry` (`check-release-consistency.mjs:9`) |
| `eval-agent-fixtures.mjs` | Scripted tool trajectories, no provider calls (`eval-agent-fixtures.mjs:61`) |
| `generate-worker-icon-assets.mjs` | Regenerates `worker/src/connector-icon-assets.js` from `codex/assets/icon.png` (`generate-worker-icon-assets.mjs:2`) |
| `hosted-consent.mjs` | One-time browser consent; stores the refresh token in 1Password (`hosted-consent.mjs:3`) |
| `smoke-batch-verify.mjs` | Live smoke for batch verification through approved calls |
| `smoke-hosted.mjs` | Post-deploy check of the hosted connector (`smoke-hosted.mjs:3`) |
| `smoke-list-tools.mjs` | Asserts the required tool names exist (`smoke-list-tools.mjs:5`) |
| `smoke-review-unapproved.mjs` | Live smoke for `review_unapproved` |
| `sync-plugin-metadata.mjs` | Syncs the plugin manifests from `package.json` (`sync-plugin-metadata.mjs:8`) |
| `test-safety-model.mjs` | Safety model check over the read and write tool lists (`test-safety-model.mjs:12`) |

## For AI agents
- Offline gates run inside `npm test`: `test:safety` and `test:eval` (`../package.json:61`, `../package.json:62`).
- `smoke:list-tools` is an offline discovery check (CI runs it with `YNAB_DISABLE_AGENT_CONFIG_FALLBACK=1`). `smoke:review-unapproved` and `smoke:batch-verify` call a real budget; `smoke:hosted` and `hosted-consent.mjs` need the hosted connector and a 1Password grant (`hosted-consent.mjs:3`).

## Dependencies
- `scripts/lib/` helpers, `../index.js`, `../package-lock.json` (`check-release-consistency.mjs:35`), `../codex/assets/`, `../worker/src/`.
- External: `@modelcontextprotocol/sdk`, `zod`, `ynab`, the 1Password `op` CLI.
