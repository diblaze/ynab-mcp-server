<!-- Parent: ../AGENTS.md -->
# lib/app-views/

## Purpose
Self-contained HTML, CSS and JS for the two read-only MCP App views, `ui://ynab/budget-overview` and `ui://ynab/transaction-review`. `lib/app-views.mjs` inlines them at runtime; they render `get_month` and `review_unapproved` output and never call a write tool.

## Key files
| File | Purpose |
|------|---------|
| `budget-overview.html` | Ready to Assign and category group rows; fetches `get_month` (`budget-overview.html:104`) |
| `transaction-review.html` | Unapproved and uncategorized transactions with an "ask Claude" button (`transaction-review.html:152`) |
| `shared.js` | Client helpers: the `h()` DOM builder and `askClaude` (`shared.js:30`, `shared.js:95`) |
| `shared.css` | YNAB-styled theme variables inlined into each view (`shared.css:1`) |

## For AI agents
- Test: `npm run test:unit`; `test/app-views.test.mjs` builds the views through `buildViewHtml`.
- Views stay read-only: the only tool call is `get_month` (`budget-overview.html:104`); `askClaude` only asks for a proposal (`shared.js:93`).
- Render tool data as text only, through `textContent` and text nodes (`shared.js:35`, `shared.js:41`).
- `/*SHARED_CSS*/`, `/*SHARED_JS*/` and `/*APP_BUNDLE*/` are replaced at runtime by `buildViewHtml` (`../app-views.mjs:56`), and `</script` is escaped so the bundle cannot break the inline tag (`../app-views.mjs:49`).

## Dependencies
- Read and inlined by `lib/app-views.mjs:31`.
- Runtime: the inlined `@modelcontextprotocol/ext-apps` bundle (`../app-views.mjs:38`); no CDN or network loads, no npm imports in the view files.
