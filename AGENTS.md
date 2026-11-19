# ynab-mcp-server

Node MCP server for YNAB. `index.js` holds the server and its tools (`createYnabServer`, `index.js:595`); `lib/` and `modules/` hold the helpers it imports.

## Coding standard
- Stack: Node ≥20 ESM (`package.json` `"type": "module"`); MCP SDK ^1.30 — Context7 `/modelcontextprotocol/typescript-sdk`; MCP Apps — `/modelcontextprotocol/ext-apps`; `ynab` ^4.5 — `/ynab/ynab-sdk-js` (API reference `/websites/api_ynab`); `zod` ^4.4 — `/colinhacks/zod`. `worker/`: Cloudflare Workers + wrangler.
- Package manager: npm (lockfiles `package-lock.json`, `worker/package-lock.json`). Never another.
- Check: `npm ci && npm ci --prefix worker && node --check index.js && npm run lint && npm test && npm run release:check && YNAB_DISABLE_AGENT_CONFIG_FALLBACK=1 npm run smoke:list-tools` (+ `npm test --prefix worker` when `worker/` changes) — green before commit. CI: `.github/workflows/ci.yml`.
- Format/lint: Biome 2.6.8 (exact devDependency) lint only, config `biome.json` (formatter off, upstream's formatting stays). Errors fail, warnings are upstream debt: don't add new ones.
- Tests: `node:test`, `test/*.test.mjs` and `worker/test/*.test.mjs`, synthetic data and the fake YNAB server only. New conditional skips need a reason string (upstream's `{ skip: !posix }` in `test/op-token-cache.test.mjs:24` predates the rule); bare skip/`.only` is a blocker.
- Style: tool input and output is dollars, internal math is integer milliunits; convert only through `dollars()`/`milliunits()` (`index.js:651`, `index.js:655`); every write goes through `preview_write_tool` and a single-use token (`lib/write-previews.mjs:51`); multi-step writes use the durable journal (`lib/write-recovery.mjs:37`).
- Language: code, comments, commits and docs English.
- Fork: `main` = upstream `oliverames/ynab-mcp-server` + our commits (ADR `docs/adr/0001-maintain-own-fork.md`). Upstream sync: branch worktree from `main`, `git merge upstream/main`, Check, PR in the fork merged with a merge commit (never rebase-merge an upstream sync, it rewrites upstream SHAs). Our own changes: rebase-merge. Layout `~/git/ynab-mcp-server/{.bare,main,<branch>}`; the live MCP runs from `main/`: never edit it directly, and after a merge run `git -C main pull --ff-only`, `npm ci` in `main/`, then `/mcp` reconnect.
- Tracking: GitHub Issues in the fork. `gh` defaults to upstream here: always pass `-R diblaze/ynab-mcp-server`. Labels `type:bug|feat|chore` on PRs; the last slice commit carries `Refs diblaze/ynab-mcp-server#n` and the PR body `Closes diblaze/ynab-mcp-server#n`.
- Don't: run `publish.sh`/`npm publish`, `smoke:hosted`, `hosted-consent` or the live `test.js` (real writes, `test.js:10`); don't deploy `worker/` (the fork has no hosted connector).

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
| `docs/adr/` | Architecture decision records (`0001-maintain-own-fork.md`) |
| `assets/` | Package icon and logo |
| `codex/` | Codex plugin manifest and the icon the worker icons are built from |
