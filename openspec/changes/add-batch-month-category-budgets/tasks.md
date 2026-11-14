# Tasks
## 1. Batch tool with month-level reads
- [x] 1.1 Write failing tests in `test/category-recovery-integration.test.mjs`. Use synthetic `fake-food`/`fake-travel` in `2026-10-01` (E32) and the `harness.write`/`injectFault` patterns (E33). Each test proves one scenario:
  - `::batch applies every item with readback` proves scenario "Batch applies every item". Items are reported in dollars.
  - `::batch preview lists current and proposed amounts` proves scenario "Preview lists every item". It asserts 0 writes.
  - `::batch rejects duplicate category at preview` proves scenario "Duplicate category refused".
  - `::batch rejects unknown category at preview` proves scenario "Unknown category refused".
  - `::batch rejects more than 50 items at preview` proves scenario "Oversized batch refused".
  - `::batch rejects malformed month at preview` proves scenario "Malformed month refused".
  - `::stale batch item aborts before any write` proves scenario "Changed item after preview". It asserts 0 writes and an empty journal.
  - `::batch stops at first failed write` proves scenario "Write failure stops the batch". It injects a fault on the `fake-food` PATCH and asserts 1 write, with both items `pending`.
  - `::batch final readback mismatch is a conflict` proves scenario "Final readback mismatch reported". It uses `persist-drift` on the `fake-food` PATCH and asserts `observed` in dollars.
  - `::partial batch resumes after new approval` proves scenario "Partial batch resumes after new approval". It injects a fault on `fake-travel`, then runs `harness.write("resume_operation", ...)`, which must complete with exactly 1 more write.
  - `::batch unchanged item is not written` proves scenario "Unchanged item skipped".
  - `::batch uses month reads only` proves scenario "Request count for a full batch". It runs 3 changed items (adds a synthetic third category to the fixture) and, counting `fake.requests` after the preview, asserts 3 `GET /months/2026-10-01`, 3 PATCH and 0 `GET .../categories/{id}`.
- [x] 1.2 Extend `persist-drift` in `test/support/fake-ynab.mjs:129-136` so it can target a month-category row (E32).
- [x] 1.3 In `lib/write-recovery.mjs`, make `run(input, options)` pass `options.readAll` to `execute` (E16):
  - In batch mode, preflight makes one `readAll` call (E11).
  - The pre-write read is skipped (E12).
  - A successful write uses the `writeStep` return value as `after`. A thrown write still calls `readStep`.
  - A final `readAll` marks a step that differs as `conflict` / `final_readback_mismatch` with `observed`.
  - `summary` passes `observed` through when set (E15).
  - The classic path stays unchanged. Verify: `cd test && node --test recovery.test.mjs category-recovery-integration.test.mjs`.
- [x] 1.4 In `index.js`, three changes:
  - Add `readMonthSteps` (one `getPlanMonth`, E25).
  - Make `writeOperationStep` return `{ budgeted: data.category.budgeted }` for `category_budget` (E10).
  - Add the `buildCategoryPlan` batch branch before `const from` (E6). It runs `endOfMonth`, rejects duplicates, makes one `getPlanMonth`, rejects a missing or deleted id, and builds E7 steps in the E8 shape.
- [x] 1.5 Add `'update_month_categories'` to `lib/write-tool-state.mjs:44` (E5). In `index.js`:
  - Add the `WRITE_TOOL_METADATA` entry (E26).
  - Register the tool next to `move_category_budget` (E27), with `categories: z.array(z.object({ categoryId: z.string(), budgeted: z.number().finite() })).min(1).max(50)`.
  - The handler calls `operationRunner.run(plan, { readAll: readMonthSteps })` and returns the summary plus `items` in dollars. Verify: 1.1 passes.
- [x] 1.6 Review fixes: final readback failure records `final_readback_unavailable` (`::batch final readback unavailable is unknown with a reason` proves scenario "Final readback unavailable"); deleted category refused at preview (`::batch rejects deleted category at preview`); resume clears a stale mismatch (`::batch resume after final mismatch reports no stale mismatch`); `observed` returned as a top-level dollars map, not in summary steps; `monthRows` shared helper.

## 2. Gating, listing, docs
- [x] 2.1 Add `"update_month_categories"` to `writeTools` in `scripts/test-safety-model.mjs:12-39` (E30). This proves scenario "Read-only session hides the batch tool". Verify: `npm run test:safety`.
- [x] 2.2 Add `"update_month_categories"` to `requiredWriteTools` in `scripts/smoke-list-tools.mjs:19` (E30).
- [x] 2.3 Update `README.md`:
  - Counts: `67 tools` becomes `68 tools`, `71` becomes `72`, `25 YNAB operations` becomes `26`, and Categories `9` becomes `10` (E31).
  - Add a row after `update_month_category`: "up to 50 categories in one month from one preview; month-level reads, stops at first failure, recover with `resume_operation`".
  - Add the tool to the durable-tools sentence at line 577.
  - Add one sentence after the limiter paragraph in Rate Limiting (line 725, E35): a batch such as `update_month_categories` needs about N+4 requests at once, so raise `YNAB_RATE_LIMIT_BURST` (for example 150) to run it unpaced.
  - Verify: `npm run release:check`.

## 3. Check
- [x] 3.1 `npm test && npm run release:check && YNAB_API_TOKEN=synthetic-smoke-token YNAB_ALLOW_WRITES=1 YNAB_DISABLE_AGENT_CONFIG_FALLBACK=1 npm run smoke:list-tools`
- [x] 3.2 This step runs on the main thread, outside the repo, and is not committed. Point the `ynab` entry in `~/.claude.json` (E34) at `node <path-to-checkout>/index.js`, keeping the env (including `YNAB_RATE_LIMIT_BURST=150`) and adding `YNAB_ALLOW_WRITES=1`. Restart and confirm `update_month_categories` is listed. On a test budget, run one batch of about 40 items (preview, then approve) and time it. It must finish within about a minute and report `completed`. Result 2026-11-14: 30 items, `completed`, 30/30 `verified_applied`, ~25 s.
