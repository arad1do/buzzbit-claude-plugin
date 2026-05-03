---
description: Show current BuzzBit X workspace tier, MCP quota usage, and recent activity.
---

# /buzzbit-status

When the user runs this slash command, do exactly the following — no extra commentary:

1. Call `get_workspace_limits`. From the response, render a compact status table:

```
BuzzBit X — Workspace Status
─────────────────────────────
Tier              <tier>
Subscription      <subscriptionStatus>
Period ends       <periodEnd, formatted as YYYY-MM-DD>

Monthly quotas    used / limit  (% used)
─────────────────────────────────────────
API calls         <used> / <limit>  (<pct>%)
Drafts            <used> / <limit>  (<pct>%)
Campaigns sent    <used> / <limit>  (<pct>%)
Social posts      <used> / <limit>  (<pct>%)
DMs sent          <used> / <limit>  (<pct>%)
```

2. Call `get_dashboard_overview` with `period: "week"`. Append:

```
Last 7 days
───────────
Revenue           $<revenue>
Orders            <orders>
New customers     <newCustomers>
AOV               $<avgOrderValue>
```

3. If any quota counter is ≥ 80% used, add a single warning line:

```
⚠ <counter> approaching limit — consider upgrading your plan.
```

4. Stop. No further explanation unless the user asks.
