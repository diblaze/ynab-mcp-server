<!-- Parent: ../AGENTS.md -->
# scripts/lib/

## Purpose
Shared helpers for the smoke and hosted scripts: an MCP client over stdio or HTTP, an interactive approval gate for write calls, the hosted OAuth 2.1 client and the 1Password token store.

## Key files
| File | Purpose |
|------|---------|
| `approved-tool-call.mjs` | Wraps tool calls behind an interactive approval prompt (`approved-tool-call.mjs:19`) |
| `hosted-cli.mjs` | Argument parsing for the hosted scripts (`hosted-cli.mjs:13`) |
| `hosted-oauth.mjs` | OAuth 2.1 client helpers: PKCE, discovery, token exchange and refresh (`hosted-oauth.mjs:3`) |
| `hosted-smoke.mjs` | `runHostedSmoke` and `runHostedConsent` (`hosted-smoke.mjs:57`, `hosted-smoke.mjs:181`) |
| `op-token-store.mjs` | 1Password storage for the consent grant (`op-token-store.mjs:6`) |
| `smoke-client.mjs` | stdio and HTTP MCP client, `parseSmokeOptions`, `withSmokeClient` (`smoke-client.mjs:114`) |

## For AI agents
- Covered by `npm run test:unit` (`test/approved-tool-call.test.mjs`, `test/hosted-smoke.test.mjs`).
- Live scripts approve through the interactive prompt only, never an environment grant (`approved-tool-call.mjs:17`).
- Secrets never appear in thrown errors (`hosted-oauth.mjs:5`); keep it so when editing.
- The stored refresh token lapses after 30 days by default and rotation does not extend it (`hosted-smoke.mjs:17`).

## Dependencies
- Used by `scripts/*.mjs` and `test/`.
- External: `@modelcontextprotocol/sdk`, Node `http`/`crypto`/`child_process`, the 1Password `op` CLI.
