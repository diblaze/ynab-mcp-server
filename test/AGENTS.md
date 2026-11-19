<!-- Parent: ../AGENTS.md -->
# test/

## Purpose
Offline `node:test` suites for the server, run from this directory. They use synthetic data and a loopback fake YNAB HTTP server (`support/fake-ynab.mjs`), never real credentials. `fixtures/agent-safety.json` feeds the scripted agent eval.

## Key files
| File | Purpose |
|------|---------|
| `unit.test.mjs` | Pure helpers of `index.js` (`unit.test.mjs:1`) |
| `integration-mcp-http.test.mjs` | MCP over HTTP against the fake YNAB server |
| `write-preview.test.mjs` | Preview token issue and claim (`lib/write-previews.mjs`) |
| `recovery.test.mjs` | Durable write journal and operation runner (`recovery.test.mjs:3`) |
| `category-recovery-integration.test.mjs` | MCP to HTTP to file-journal recovery, synthetic (`category-recovery-integration.test.mjs:1`) |
| `undo-safety-integration.test.mjs` | Undo safety regression fixtures (`undo-safety-integration.test.mjs:1`) |
| `matched-links.test.mjs` | Matched transaction links |
| `cache.test.mjs` | Entity cache and field projections (`cache.test.mjs:3`) |
| `cache-integration.test.mjs` | Cache through MCP and the fake server |
| `analytics.test.mjs` | `lib/analytics.mjs` on synthetic data (`analytics.test.mjs:3`) |
| `analytics-tools.test.mjs` | Analytics tools through MCP |
| `category-suggestions.test.mjs` | `modules/category-suggestions.mjs` (`category-suggestions.test.mjs:4`) |
| `app-views.test.mjs` | App view HTML through `buildViewHtml` (`app-views.test.mjs:5`) |
| `error-reporting.test.mjs` | Sentry events carry no financial data (`error-reporting.test.mjs:1`) |
| `worker-bootstrap.test.mjs` | Worker-safe module bootstrap of `index.js` (`worker-bootstrap.test.mjs:5`) |
| +3 more | `approved-tool-call.test.mjs`, `hosted-smoke.test.mjs`, `op-token-cache.test.mjs` |

## For AI agents
- Run `npm run test:unit` from the repo root (`cd test && node --test --test-concurrency=2`, `../package.json:43`); `pretest:unit` checks dependencies first. Paths inside tests are relative to `test/` (`../lib/...`).
- Synthetic data only (`support/fake-ynab.mjs:1`, `analytics.test.mjs:5`); never add real budget data.

## Dependencies
- `../lib/`, `../modules/`, `../index.js`, `../scripts/lib/approved-tool-call.mjs`.
- External: `node:test`, `@modelcontextprotocol/sdk`, `@sentry/node`, `zod`.
