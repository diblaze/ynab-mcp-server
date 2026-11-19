# 0001. Run and maintain our own fork of the YNAB MCP server

- Status: accepted
- Date: 2026-11-19

## Context
We run this server as the `ynab` MCP for day-to-day budgeting, and we depend on `update_month_categories` to set a month's budget from one preview and one approval. Upstream closed the PR that added it (oliverames/ynab-mcp-server#53) unmerged on 2026-11-17. The owner retired the project and gave two reasons: YNAB is releasing an official MCP server, and YNAB is changing its API policy for third-party MCP access.

On 2026-11-19 we could confirm neither reason:
- https://api.ynab.com/ lists the API Starter Kit and Zapier as the only official integrations.
- No policy change is published. Personal access tokens work as before (200 requests per hour).
- Unverified: one third-party project reports that YNAB stopped listing third-party MCP servers in its "Works with YNAB" directory on 2026-11-18 (https://github.com/mathbeal/avenir-mcp/pull/212, an open PR opened 2026-11-17). We have not confirmed this on YNAB's own pages.

## Decision
`diblaze/ynab-mcp-server` is the source we run. Its `main` is upstream 6.2.1 (`ed3435f`) plus our own commits, starting with `update_month_categories` (`cc29423`, fork PR #2). The MCP runs from the `main/` worktree. If upstream still publishes a release, we merge it in by hand on a branch and land it through a PR in the fork with a merge commit, following the `## Coding standard` section of `AGENTS.md`. The fork publishes nothing to npm and deploys no hosted worker.

## Alternatives rejected
- **Switch to another community server** (calebl/ynab-mcp-server, klauern/mcp-ynab and others). None of them has the preview-token write model or the durable write journal (`lib/write-previews.mjs`, `lib/write-recovery.mjs`), which we rely on for every write to real money. We would also have to rebuild the batch tool on a new codebase.
- **Wait for YNAB's official MCP.** It has not been announced, and we don't know what it will cover. Batch month budgeting and preview-before-write may never be part of it. Meanwhile we need a working MCP today.
- **Run the upstream npm package unchanged.** It lacks the batch tool, and the owner said on #53 that the project "won't get further releases" (6.2.1 still shipped on 2026-11-19).

## Consequences
- The batch tool, fixes and upgrades are ours from now on, and nobody else reviews them. Upstream releases are not expected; any that come still have to be merged by hand, and each fork-only change makes that harder, so fork changes stay small and in their own files where possible.
- Biome now lints the fork (`biome.json`, errors fail CI). This is a small divergence from upstream.
- The hosted worker, the plugin manifests and `publish.sh` stay in the tree but are unused. Deleting them would buy nothing today and would conflict with any late upstream release; remove them once upstream has clearly stopped.
- If YNAB restricts personal-token API use, the fork stops working along with every other community server.

## Reversal
Revisit if YNAB ships an official MCP server that covers batch month budgeting and confirms writes before applying them, or if YNAB's API terms forbid MCP use with a personal access token. Undoing is cheap: point `~/.claude.json` at the replacement and archive the fork. The work lost is the batch tool (one commit) and this documentation.
