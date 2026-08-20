# Crawl budgets

This skill spends the human's own tool-call and read quota, not the server's.
The server caps plan size on its own (entry counts, byte totals); these
budgets are separate and smaller, because a crawl that reads everything before
a human sees a plan defeats the point of proposing first. Stop crawling the
moment a budget is spent, rank what you have, and move on to the next step.

## The five budgets

- **Local scan budget: 2,000 entries.** The number of local files and folders
  you list for structure and metadata (path, inferred title, modified time)
  before ranking. This never reads a file body.
- **Local content-read budget: 200 files.** The number of local files you read
  full content for. Read only the files the ranking selected — never the
  whole scanned set. This budget is always smaller than the scan budget.
- **Browse-call budget: 20 calls total, shared across every connected account
  in this crawl.** The server keys `artifactbridge_browse_connected_source`
  calls on workspace + turn (or, for an agent token, workspace + token — it
  does not reset between turns) (`CONNECTED_SOURCE_BROWSE_PLAN_CALL_LIMIT`),
  not on the account you pass in — every account you browse in this crawl
  draws from the same 20-call pool, not 20 calls each. Before you browse the first
  account, divide the 20 calls across the accounts the human named (for
  example, 2 accounts get up to 10 calls each; 4 accounts get up to 5 each).
  If one account needs more, take it from an account that needs less rather
  than exceeding the pool.
- **Workspace daily browse budget: 50 calls per workspace per day.** A second,
  larger cap the server enforces across every crawl and every plan in the
  workspace that day (`CONNECTED_SOURCE_BROWSE_WORKSPACE_DAILY_CALL_LIMIT`).
  It is shared with other crawls and other agents, not just this one. Stop
  browsing for the day if you hit it, even if the 20-call pool for this crawl
  is not spent.
- **Proposed-item budget: 300 items total.** The number of items you propose
  across the local plan and every connected plan combined, in one bundle.
  Reserve a share of the 300 for each section the human named before you
  start reading or browsing, so one large section cannot consume the whole
  budget and leave the others at zero. Rank and keep the best items within
  each section's share; report the rest as "found but not proposed," with a
  count, not a list of bodies.
- **Total metadata operations budget: 2,020 calls per crawl.** The sum of
  local scan entries (2,000) and browse calls (20, shared across accounts).
  This is a cross-check on the two counters above, not an independent cap: if
  you are about to make a scan-listing or browse call that would push the
  running total of both counters combined past 2,020, stop crawling instead —
  it means one of the counters above was miscounted, since honoring both of
  them individually can never reach this total any other way.

## What to do when a budget runs out

- Local scan budget spent: stop listing. Rank what you found. Tell the human
  the directory has more entries than the scan budget covered, with a count.
- Local content-read budget spent: propose the files already read. Tell the
  human how many ranked candidates were left unread.
- Browse-call budget spent (`connected_source_browse_plan_budget_exhausted`):
  stop browsing every connected account, not just the current one — the pool
  is shared. Move to planning with what you found. Report which accounts were
  covered and which were not.
- Workspace daily browse budget spent
  (`connected_source_browse_workspace_budget_exhausted`): stop browsing for
  the rest of the day. Move to planning with what you found, and tell the
  human the workspace-wide daily browse limit was reached.
- Proposed-item budget spent: stop adding items to any plan. Report the total
  candidate count you ranked against the count you proposed.
- Total metadata operations cross-check tripped: stop crawling — no more scan
  listing, no more browsing. Move straight to planning and bundling with
  whatever was found before the cross-check. Report the operation count
  against the cross-check.
- Every case above ends the same way: create the bundle from what you have.
  A partial crawl still returns one bundle review URL; it never aborts
  silently.

## Why these numbers

- The browse-call budget matches the server's `CONNECTED_SOURCE_BROWSE_PLAN_CALL_LIMIT`
  (20) so this skill never wastes calls a human's next crawl would need. It is
  a shared pool, not a per-account allowance, because the server tracks it
  that way.
- The workspace daily browse budget matches the server's
  `CONNECTED_SOURCE_BROWSE_WORKSPACE_DAILY_CALL_LIMIT` (50) so this skill
  reports the same limit the server will enforce, instead of discovering it
  only from a failed call.
- The proposed-item budget (300) keeps one bundle review readable in one
  sitting. A human reviewing more than 300 proposed items at once is not
  reviewing them; ask for a narrower crawl instead of raising this budget.
- The local scan and content-read budgets are separate because ranking on
  metadata is cheap and content reads are not. Scan wide, read only what you
  will actually propose.
- The total metadata operations cross-check exists to catch a counting error
  in the two sub-budgets above, not to add a third independent limit: honoring
  the local scan budget (2,000) and the browse-call budget (20) individually
  already keeps the combined total at or under 2,020.
