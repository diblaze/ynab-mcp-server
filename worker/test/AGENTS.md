<!-- Parent: ../AGENTS.md -->
# worker/test/

## Purpose
`node:test` suites for the worker: origin rules, branding, icons, and the handler and credential behaviour. They run on plain Node, without the Cloudflare runtime.

## Key files
| File | Purpose |
|------|---------|
| `worker.test.mjs` | Handler, credential and transient-state behaviour (`worker.test.mjs:254`) |
| `connector-origin.test.mjs` | Origin validation (`connector-origin.test.mjs:12`) |
| `public-branding.test.mjs` | Canonical origin on landing and policy pages (`public-branding.test.mjs:10`) |
| `connector-icons.test.mjs` | Icons public and cacheable without OAuth (`connector-icons.test.mjs:18`) |

## For AI agents
- Run from `worker/`: `npm test` (`../package.json:10`).
- Tests import `../src/*` directly; no test loads `ynab-mcp.js` or the Durable Object classes, so cover new logic in a plain module.
- Brand SVG bytes are pinned (`../src/brand-assets.js:60`); a test failure there means the bytes changed.

## Dependencies
- `../src/*`; `worker.test.mjs` also imports `../../lib/write-recovery.mjs` (`worker.test.mjs:1650`) and `../../test/support/fake-ynab.mjs` (`worker.test.mjs:1714`).
- External: `@cloudflare/workers-oauth-provider` (`connector-origin.test.mjs:174`), and through `../src` also `hono` and `agents`; `node:test` and other Node built-ins.
