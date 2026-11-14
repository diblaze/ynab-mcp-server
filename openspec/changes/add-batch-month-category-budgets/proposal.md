# Proposal

Status: done
Issue: diblaze/ynab-mcp-server#1

## Why
Setting a month's budget across many categories needs one `preview_write_tool` call and one approval per category, because `update_month_category` writes a single category. Running those writes through the existing durable runner would take 4N+2 requests, paced by the local limiter at about 19 s each. For a typical month of ~40 categories that is about 45 minutes. One approval should set up to 50 categories in about a minute, with the same safety.

## What Changes
- A new write tool, `update_month_categories`. The input is `budgetId`, `month` (first of month) and 1–50 items `{categoryId, budgeted}` in dollars.
- One preview lists each item's current and proposed amount. One approval runs the whole batch.
- Execution reads the whole month instead of each item. It sends at most N+4 requests: a month read for the stale check, a month read for the preflight, one PATCH per changed item, a final month read, and one item read only after a failed write.
- Speed comes from configuration, not limiter code: with `YNAB_RATE_LIMIT_BURST` raised (for example 150), a full batch runs without local pacing. The README rate-limiting section says so.
- Execution stops at the first failure without retrying. It reports per-item status, outcome and dollars, plus the observed amount on a mismatch. `resume_operation` recovers a partial batch.
- If any item changed after the preview, the batch is refused before any write. Unchanged items are recorded as `no_change` and get no PATCH.
- The tool is hidden unless writes are enabled, like every write tool.

## Capabilities
### New Capabilities
- `month-category-batch-budget`: sets budgeted amounts for up to 50 categories in one month from one approved preview, using month-level reads, with stop-on-failure and recovery.
### Modified Capabilities
None. `openspec/specs/` is empty.

## Impact
- `lib/write-recovery.mjs`: an optional `readAll` mode for `run()`, and `observed` in the summary when set. The classic path is unchanged.
- `index.js`:
  - `readMonthSteps`
  - `writeOperationStep` returns the PATCH projection
  - a batch branch in `buildCategoryPlan`
  - a `WRITE_TOOL_METADATA` entry and the registration
- `lib/write-tool-state.mjs`: the tool name joins the category-plan list.
- Tests:
  - `test/category-recovery-integration.test.mjs`
  - `test/support/fake-ynab.mjs`: `persist-drift` for month categories
  - `scripts/test-safety-model.mjs`
  - `scripts/smoke-list-tools.mjs`
- `README.md`: counts 67→68, 71→72, 25→26, Categories 9→10, a table row, and one sentence in Rate Limiting about raising `YNAB_RATE_LIMIT_BURST` for batch writes.
- The hosted worker gets the tool from the shared factory, so no worker change is needed. No limiter code changes, new dependencies, config keys or migrations.
