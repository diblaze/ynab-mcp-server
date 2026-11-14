# Design
## Context
- E1 `index.js:2300-2305`: `api.categories.updateMonthCategory(...)` followed by `return ok(formatCategory(data.category))`. One category per call, and the PATCH response carries the saved category.
- E2 `docs/openapi-spec.yaml:875-881`: `/plans/{plan_id}/months/{month}:` has `get:` / `operationId: getPlanMonth` and no write verb. `:409,454-460`: `patch:` is "Update a category for a specific month. Only `budgeted` (assigned) amount can be updated." There is no batch write endpoint, so N items need N PATCHes. `:2550-2562`: `SaveCategoryResponse` contains `category`.
- E3 `docs/api-documentation.md:572-574`: "An access token may be used for up to **200 requests per hour**. The limit is enforced within a rolling window."
- E4 `lib/write-tool-state.mjs:74-75`: the `update_month_category` preview reads one category.
- E5 `lib/write-tool-state.mjs:44-46`: `['merge_category','retire_category','move_category_budget'].includes(name)` routes these tools to `buildCategoryPlan`, and the preview carries every step's before and after.
- E6 `index.js:1751-1755`: `if (!journal) throw ...; const from = input.fromCategoryId || input.categoryId; ... if (from === to) throw`. The batch branch has to come before line 1753.
- E7 `index.js:1780`: `steps.push({ id: \`budget:${month}:${from}\`, kind: "category_budget", target: { month, categoryId: from }, before: { budgeted }, after: { budgeted } })`.
- E8 `index.js:1785`: `return { operationId: input.operationId || fingerprint({ tenantId, sessionId, bid, tool, intent, steps }), ...`.
- E9 `index.js:1788-1793`: `executeCategoryPlan`. Its note ("Source categories still exist…") does not fit a batch.
- E10 `index.js:1736-1748`: `readOperationStep` makes one `getMonthCategoryById` per step. `writeOperationStep` PATCHes and returns nothing.
- E11 `lib/write-recovery.mjs:257-273`: preflight reads every pending step with `readStep` before the first write.
- E12 `lib/write-recovery.mjs:278-280`: a second `readStep` runs before each write. `:295-298`: `writeStep`, then a third `readStep` for readback. So the runner makes 3 reads and 1 write per item, and as-is it cannot meet N+3.
- E13 `lib/write-recovery.mjs:303-314`: the result is classified as verified, `partial` or `conflict`, and the runner returns at the first failure without retrying.
- E14 `lib/write-recovery.mjs:263-265`: before equal to after gives `no_change`, and no write is sent.
- E15 `lib/write-recovery.mjs:162`: the summary `steps` carry `{ id, kind, status, attempts, outcome }`.
- E16 `lib/write-recovery.mjs:321-323`: `run(input)` and `resume(scope)`, both through `execute`.
- E17 `lib/write-recovery.mjs:143-145`: duplicate targets throw `INVALID_PLAN`, but only at run time, after the approval.
- E18 `index.js:1826-1828`: `previews.claim`, then `prepareWrite`, then `validate(...)`. Execution re-runs `buildCategoryPlan` against fresh state. `lib/write-previews.mjs:151`: "Stale preview: … No write was attempted".
- E21 `index.js:1387-1398`: the limiter defaults to 190 requests per hour with burst 10, `refillMs = 3600000 / requestsPerHour` (about 18.9 s), and it waits in a loop. `const burst = Math.max(1, Math.floor(envNumber("YNAB_RATE_LIMIT_BURST", 10, { min: 1 })))`: the burst is tunable by env with no upper bound, and `tokens = burst` at start. `README.md:645-646` documents `YNAB_RATE_LIMIT_PER_HOUR` and `YNAB_RATE_LIMIT_BURST`.
- E23 `index.js:1702-1715` and `:2017`: `parseToolExecuteInput` validates the schema inside `preview_write_tool`. `index.js:2811`: the `.min(1).max(50)` array-cap pattern.
- E24 `index.js:1206-1209`: `endOfMonth` throws the month format error.
- E25 `index.js:2552`: `api.months.getPlanMonth(resolveBudgetId(budgetId), month)`.
- E26 `index.js:1871-1898` `WRITE_TOOL_METADATA`, and `:1987-1989` `if (!writesEnabled()) { return undefined; }`. `index.js:1821` adds `previewToken` and `confirmed`.
- E27 `index.js:4092-4095` registers `move_category_budget`. `:4105-4108`: `resume_operation` runs `operationRunner.resume(...)`.
- E28 `worker/src/ynab-mcp.js:2-4` (shared factory) and `worker/src/hosted-options.js:12` (`journal: createKvJournal(...)`). The worker needs no change.
- E29 `rg update_month_category` over `.claude-plugin codex .mcp.json .hermes-plugin .antigravity-plugin worker` finds 0 hits. No plugin metadata lists tools.
- E30 `scripts/test-safety-model.mjs:12-39` lists `writeTools`, and lines 248-253 assert `confirmed`/`previewToken`. `scripts/smoke-list-tools.mjs:18-20` has `requiredWriteTools`.
- E31 `README.md:13,332,338` give the counts 67/71/25 and Categories 9. `scripts/check-release-consistency.mjs:66-71` checks "N tools".
- E32 `test/support/mcp-harness.mjs:13`: `process.env.YNAB_RATE_LIMIT_PER_HOUR = "0"`. `test/support/fake-ynab.mjs:122` records every request in `this.requests`, `:195-204` serves the month and month-category routes, and `:129-136` applies `persist-drift` to transactions only. `:10-11` define synthetic `fake-food` 100000 and `fake-travel` 20000.
- E33 `test/category-recovery-integration.test.mjs:26-35,105-114` hold the partial and stale patterns.
- E35 `README.md:725`: the Rate Limiting paragraph ("applies a client-side limiter at 190 requests per hour with a burst of 10 by default … durable workflows can read each affected entity or step").
- E34 `~/.claude.json:4366` (outside the repo): `"github:oliverames/ynab-mcp-server"`.

## Goals / Non-Goals
**Goals:**
- One preview and one approval for up to 50 month-category budgets, finishing in about a minute.
- At execution, at most N+4 YNAB requests (N+3 without failures), so a raised burst runs a full batch unpaced. The safety model stays the same: fingerprint stale check, preflight, stop at the first failure, journal, and `resume_operation`.

**Non-Goals:**
- A batch path for `resume_operation`. Resume of a batch uses the classic per-step runner (E16, E27): the resume preview reads N steps, execution re-prepares with N reads, then runs the classic per-step pre-read, write and post-read. A 50-item batch that fails early can exceed YNAB's 200/h and pace or 429 for over an hour. Reopen when a real partial batch leaves many pending items.
- Limiter code changes (reservation, per-tool bypass). Speed comes from `YNAB_RATE_LIMIT_BURST` (E21). Reopen when a raised burst is not acceptable for a host.
- Multiple months in one batch. Reopen when a caller needs to plan across months.
- Dollar amounts inside the preview plan. It stays in milliunits like `move_category_budget` (E5); the result reports dollars. Reopen when an assistant misreads them.
- The upstream PR.

## Decisions
- **Reuse the durable runner and add a month-read mode.** `run(input, { readAll })` changes the runner at three points, and only when `readAll` is passed:
  1. Preflight makes one `readAll` call instead of N `readStep` calls (E11).
  2. The pre-write `readStep` is skipped. Preflight just verified `before`, and the plan was re-fingerprinted moments earlier (E12, E18).
  3. After a successful write, the PATCH response projection returned by `writeStep` is the per-item check. A thrown write still gets a fresh `readStep`, so a failure costs 1 extra request and the E13 classification is unchanged.

  After the loop, one final `readAll` compares every step with its `after`. A difference makes the step `conflict`, with outcome `final_readback_mismatch`; the observed values come back as a top-level `observed` map from `summary(entry, extra)`, converted to dollars in `items[]`, never persisted in the journal. A failed final read gives `unknown` with `outcome: final_readback_unavailable`.

  Routing around the runner lost: a second executor would duplicate the journal schema that `resume_operation` and `list_operations` read (E16, E27). Other tools never pass `readAll`, so their behavior is unchanged.
- **`readMonthSteps(steps, { budgetId })` in `index.js`** makes one `getPlanMonth` call (E25) and returns `Map<step.id, { budgeted: row?.budgeted ?? null }>`. A `null` never equals `after`, so a missing row becomes a conflict rather than throwing during canonicalization.
- **`writeOperationStep` returns `{ budgeted: data.category.budgeted }`** for `category_budget` (E1, E2, E10). Classic mode ignores the return value.
- **The batch branch in `buildCategoryPlan`** goes before line 1753 (E6). It runs `endOfMonth` (E24), rejects a duplicate `categoryId` (the runner only catches duplicates after approval, E17), and makes one `getPlanMonth` call. A missing or deleted id throws. Items become E7 steps and the result has the E8 shape. Because the branch runs at both preview and execute (E18), the stale fingerprint uses the month read.
- **Speed via `YNAB_RATE_LIMIT_BURST`, documented in the README env table** (`README.md:645-646`, E21). With burst ≥ N+4, the bucket starts full and a batch of 50 (at most 54 requests) runs unpaced. No limiter code changes. One README sentence in Rate Limiting (E35) points batch writes at a higher burst. A reservation in `secureFetchRequest` lost: the user rejected it as code for what configuration already does.
- **Tool:** `update_month_categories`, `categories: z.array(...).min(1).max(50)` (E23), with metadata `{ destructiveHint: false, idempotentHint: true }` (E26). The handler returns the runner summary plus `items`: categoryId, before and after in dollars, status, outcome, and observed when present. The runner returns mismatch values as a top-level `observed` map via `summary(entry, extra)`, never in summary steps or the journal (E15).

## Risks / Trade-offs
- **Quota consumption.** A 50-item batch uses up to 55 requests: 1 for the preview, plus up to 54 at execution. That is about 29 % of YNAB's 200 per hour (E3), sent at once.
- **A burst of 150 lets every tool send up to 150 requests back to back.** That is not limited to batches. With other clients on the same token (a bank importer, another MCP host), YNAB can answer 429. A request that gets 429 is retried after `Retry-After` (README:648). That keeps a batch correct but can push it far past a minute.
- **Without a raised burst**, a batch of 40 still makes N+4 requests, and after the first 10 they are paced at about 19 s each, roughly 15 minutes. That is still about 3x fewer requests than 40 separate calls.
- **Accepted overwrite window.** Batch mode skips the per-item pre-write read. An outside edit to item k between the preflight month read and k's PATCH is overwritten and not reported, because the final readback then sees `after`. Accepted: the window is seconds, and the preview fingerprint and the preflight catch earlier drift.
- **The one-minute target is not measured.** About 50 sequential PATCHes plus 2N+2 journal saves are expected to fit; YNAB latency is unmeasured. The local smoke step in 3.2 measures it.
- **Not atomic.** A partial batch leaves earlier items applied until `resume_operation` runs (E13).

## Open Questions
- Item cap? Recommend 50 (a typical month is about 40 categories, and a full batch, at most 54 requests, fits inside a burst of 150). Raising it changes "Oversized batch refused".
- Partial-failure semantics? Recommend stop at the first unverified item (E13). The report reads `status: partial|conflict|unknown`, `recovery_required: true`, and `items[]` with status `applied|pending|conflict|unknown`, outcome, and, on a mismatch, `observed` in dollars. Items after the failure stay `pending` and are never sent.
- Skip zero-change items? Recommend keeping them in the plan. They are reported `no_change` with no PATCH (E14), so the preview and the report list every requested item.
- Tool name? Recommend `update_month_categories` (plural of `update_month_category`, like `update_transaction`/`update_transactions`, E26).
- Should `resume_operation` of a batch also use the month-read path? Recommend not now (Non-Goal). Without it, resuming a batch with many pending items costs about 2N reads plus 3 requests per pending item and can exceed the 200/h limit. Saying yes adds one ternary in the resume handler and one test.
