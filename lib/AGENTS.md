<!-- Parent: ../AGENTS.md -->
# lib/

## Purpose
Server-side helper modules behind `index.js`: write previews and the durable write journal, deterministic money analytics, a per-session entity cache, output field projection, Sentry error scrubbing, a 1Password token cache and the read-only MCP App view resources.

## Key files
| File | Purpose |
|------|---------|
| `analytics.mjs` | Income, spending, trend and forecast math on integer milliunits (`analytics.mjs:71`) |
| `app-views.mjs` | Registers the read-only `ui://ynab/*` view resources (`app-views.mjs:68`) |
| `entity-cache.mjs` | Per-server TTL cache keyed by tenant and session, with invalidation (`entity-cache.mjs:9`) |
| `error-reporting.mjs` | Optional Sentry reporting reduced to type, fixed message and tags (`error-reporting.mjs:97`) |
| `field-projections.mjs` | `lean`/`full` output field filtering per resource (`field-projections.mjs:13`) |
| `instrument.mjs` | Side-effect import that starts Sentry before `index.js` loads the rest (`instrument.mjs:5`) |
| `op-token-cache.mjs` | Mode-600 cross-process cache for a 1Password-resolved token, 10 min TTL (`op-token-cache.mjs:51`) |
| `write-previews.mjs` | Issues and claims single-use, fingerprint-bound preview tokens (`write-previews.mjs:24`) |
| `write-recovery.mjs` | Durable journal and operation runner for multi-step writes (`write-recovery.mjs:179`) |
| `write-tool-state.mjs` | Read-only pre-write state and plan builder shared by preview and execute (`write-tool-state.mjs:2`) |

## For AI agents
- Test: `npm run test:unit` (`cd test && node --test`); `write-tool-state.mjs` and `instrument.mjs` have no direct test.
- Money is integer milliunits; `analytics.mjs:15` throws on non-safe integers and `analytics.mjs:4` converts only for output.
- A preview token is a capability, not consent: `write-previews.mjs:51` rejects missing, expired, reused or stale tokens and `claim()` deletes on read (`write-previews.mjs:48`).
- Multi-step writes need the durable journal: `write-recovery.mjs:37` throws `JOURNAL_UNAVAILABLE` without it, and financial calls must not run inside a retryable journal callback (`write-recovery.mjs:2`).

## Dependencies
- Imported by `index.js` (lines 5-26), `test/` and `worker/test/worker.test.mjs:1650`; nothing in `scripts/`, `modules/` or `worker/src` imports `lib/`.
- External: `@sentry/node` (optional, `error-reporting.mjs:17`), `@modelcontextprotocol/ext-apps` (`app-views.mjs:10`); otherwise Node built-ins.
