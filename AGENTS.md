# ynab-mcp-server

Node MCP server for YNAB. `index.js` holds the server and its tools (`createYnabServer`, `index.js:595`); `lib/` and `modules/` hold the helpers it imports.

## Map
| Directory | Purpose |
|-----------|---------|
| [`lib/`](lib/AGENTS.md) | Write previews and journal, analytics, cache, projections, error reporting, view registration |
| [`lib/app-views/`](lib/app-views/AGENTS.md) | Read-only MCP App views (budget overview, transaction review) |
| `modules/` | `category-suggestions.mjs`: explained category suggestion tools (`index.js:13`) |
| [`scripts/`](scripts/AGENTS.md) | Release and build tooling, offline safety and eval gates, live smoke checks |
| [`scripts/lib/`](scripts/lib/AGENTS.md) | MCP client, approval gate, hosted OAuth and 1Password helpers for the scripts |
| [`test/`](test/AGENTS.md) | Offline `node:test` suites on synthetic data and a fake YNAB server |
| [`worker/`](worker/AGENTS.md) | Cloudflare Worker that hosts the server with YNAB OAuth |
| [`worker/src/`](worker/src/AGENTS.md) | Worker runtime: OAuth router, Durable Object MCP agent, web flow |
| [`worker/test/`](worker/test/AGENTS.md) | Worker `node:test` suites |
| [`docs/`](docs/AGENTS.md) | Design notes, API reference, release notes, privacy policy |
| `assets/` | Package icon and logo |
| `codex/` | Codex plugin manifest and the icon the worker icons are built from |
