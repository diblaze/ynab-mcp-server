<!-- Parent: ../AGENTS.md -->
# worker/

## Purpose
Cloudflare Worker that hosts the root MCP server remotely, signing each user in through YNAB OAuth. Holds the Wrangler config (KV, Durable Objects, migrations, vars), its own npm package, the runtime in `src/` and the tests in `test/` (`README.md:1`).

## Key files
| File | Purpose |
|------|---------|
| `.dev.vars.example` | Names of the local dev variables (connector URL, YNAB OAuth client, encryption keys); no values (`.dev.vars.example:3`) |
| `README.md` | Hosting guide: deploy steps and verification (`README.md:111`, `README.md:245`) |
| `package-lock.json` | npm lockfile for the worker package (`package-lock.json:4`) |
| `package.json` | Scripts `dev`, `deploy`, `test`, `types` and dependencies (`package.json:7`) |
| `wrangler.jsonc` | Worker config: entry, KV, Durable Objects, migrations, vars (`wrangler.jsonc:5`, `wrangler.jsonc:22`) |

## For AI agents
- Test from `worker/`: `npm test` (`node --test test/*.test.mjs`, `package.json:10`); `wrangler dev` / `wrangler deploy` for runtime and deploy.
- Secrets are set with `wrangler secret put` only, never written into config (`wrangler.jsonc:47`).
- Durable Object migrations are tagged `v1`–`v3` (`wrangler.jsonc:29`): add a new tag, never rewrite an existing one.
- Deploy is `npx wrangler deploy` followed by `npm run smoke:hosted` (`README.md:125`, `README.md:128`).

## Dependencies
- `src/` imports the root factory `../../index.js` (`src/ynab-mcp.js:7`); nothing in `worker/src` imports `lib/`.
- External: wrangler 4.147.0 (`package.json:19`), `nodejs_compat` runtime flag (`wrangler.jsonc:10`).
