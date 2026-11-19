<!-- Parent: ../AGENTS.md -->
# docs/

## Purpose
Design notes, API reference, release notes, upstream's privacy policy and dated worklogs. Nothing here runs; the code is the source of truth for behaviour.

## Key files
| File | Purpose |
|------|---------|
| `write-recovery.md` | Recoverable multi-step writes (`write-recovery.md:1`) |
| `testing-harness.md` | The synthetic MCP and HTTP safety harness (`testing-harness.md:1`) |
| `read-cache.md` | Read cache design (`read-cache.md:1`) |
| `mcp-app-views.md` | MCP App views (`mcp-app-views.md:1`) |
| `category-suggestions.md` | Explained category suggestions (`category-suggestions.md:1`) |
| `analytics-improvements.md` | Analytics improvements (`analytics-improvements.md:1`) |
| `hosted-oauth-connector.md` | Hosted OAuth connector pattern (`hosted-oauth-connector.md:1`) |
| `domain-migration.md` | Hosted connector domain migration (`domain-migration.md:1`) |
| `api-endpoints.md` | YNAB API endpoint reference (`api-endpoints.md:153`) |
| `api-documentation.md` | YNAB API reference, from authentication on (`api-documentation.md:11`) |
| `openapi-spec.yaml` | YNAB OpenAPI 3.1.1 contract (`openapi-spec.yaml:1`) |
| `privacy.md` | Upstream's privacy policy for the hosted connector (`privacy.md:1`) |
| `improvement-coverage.md` | Approved improvement coverage (`improvement-coverage.md:1`) |
| `claude-connector-submission.md` | Claude Connectors Directory submission draft (`claude-connector-submission.md:1`) |
| `release-6.0.0.md` | Release notes 6.0.0 (`release-6.0.0.md:1`) |
| +3 more | `release-6.0.1.md`, `YNAB-AUDIT-DELTA-2026-07-15.md`, `YNAB-COMPLETE-WORKLOG-2026-07-15.md` |

## For AI agents
- No script checks these files; treat `api-endpoints.md` and `openapi-spec.yaml` as reference, not as the source of behaviour.
- `YNAB-*-2026-07-15.md` and `archive/` are dated snapshots; do not edit them as current state.

## Dependencies
- None executable.
