<!-- Parent: ../AGENTS.md -->
# worker/src/

## Purpose
Worker runtime. `index.js` wraps `/mcp` and `/sse` in an OAuth 2.1 provider, `ynab-mcp.js` is the per-session Durable Object MCP agent built on the root server factory, and `ynab-handler.js` serves the landing, consent, YNAB OAuth, privacy and deletion pages. The rest are origin checks, encrypted token storage, credential and transient-state Durable Objects, and brand assets.

## Key files
| File | Purpose |
|------|---------|
| `index.js` | Entry: `OAuthProvider` around `/mcp` and `/sse`, default handler `YnabHandler` (`index.js:19`) |
| `ynab-mcp.js` | `YnabMCP extends McpAgent`, the per-session MCP agent (`ynab-mcp.js:11`) |
| `ynab-handler.js` | Hono web flow: landing, consent, OAuth callback, token persistence (`ynab-handler.js:612`) |
| `ynab-oauth.js` | YNAB OAuth URL, AES-GCM token record encryption and storage (`ynab-oauth.js:119`) |
| `credential-store-core.js` | Testable credential Durable Object protocol, one object per YNAB user (`credential-store-core.js:89`) |
| `oauth-credentials.js` | `OAuthCredentials` Durable Object (`oauth-credentials.js:4`) |
| `oauth-transient-state.js` | `OAuthTransientState` Durable Object (`oauth-transient-state.js:6`) |
| `transient-state.js` | Atomic one-time consume of transient records (`transient-state.js:25`) |
| `connector-origin.js` | Accepted-origin rules and the origin-scoped OAuth router (`connector-origin.js:37`) |
| `grant-replacement.js` | Captures and revokes same-origin grants (`grant-replacement.js:14`) |
| `mcp-origin.js` | Rejects untrusted browser Origins before OAuth (`mcp-origin.js:29`) |
| `mcp-session-isolation.js` | Binds the MCP session id to the grant (`mcp-session-isolation.js:25`) |
| `hosted-options.js` | Hosted server options with the per-call token getter (`hosted-options.js:3`) |
| `brand-assets.js` | Connector origin, server info and resource metadata (`brand-assets.js:82`) |
| `pages.js` | Static landing and consent pages (`pages.js:74`) |
| +3 more | `brand-assets-png.js`, `connector-icon-assets.js` (generated), `response-security.js` |

## For AI agents
- Test from `worker/`: `npm test`; tests import these modules directly, not the Durable Object classes.
- Never derive an accepted origin from Host, Origin or forwarded headers (`connector-origin.js:2`).
- Financial results and previews stay in memory; recovery uses only the encrypted journal (`ynab-mcp.js:12`), and token records are encrypted before storage (`ynab-oauth.js:119`).
- `connector-icon-assets.js` is generated (`connector-icon-assets.js:1`): rebuild with `npm run build:worker-icons` at the repo root, do not hand-edit.

## Dependencies
- Root factory `../../index.js` (`ynab-mcp.js:7`).
- External: `@cloudflare/workers-oauth-provider` (`index.js:6`), `agents/mcp` (`ynab-mcp.js:6`), `hono` (`ynab-handler.js:7`), `cloudflare:workers`.
