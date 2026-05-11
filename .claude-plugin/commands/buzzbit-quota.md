---
name: buzzbit-quota
description: Show the current BuzzBit X plan tier and quota usage for the connected workspace.
---

# /buzzbit-quota

Reports the merchant's current MCP quota usage by category:
- API calls
- Drafts created
- Campaigns sent
- Social posts published
- DMs sent

…against their plan tier's ceilings.

## What I'll do

1. Call **`get_workspace_limits`** to fetch the tier, period boundaries, and quota counters.
2. Render a compact table: counter / used / limit / percent.
3. Highlight any counter ≥ 80% (warning) or ≥ 95% (critical).
4. If any counter is near the limit, suggest concrete actions:
   - Upgrade tier (link to `/settings/billing`)
   - Reduce the planned write volume
   - Wait for period reset (show the date)

## Example output

```
Plan: GROWTH
Period resets: 2026-06-11 (in 31 days)

Counter                  Used    Limit   Used %
─────────────────────── ────── ───────── ───────
API calls               1,234   5,000     25%
Drafts created            12     100      12%
Campaigns sent             8      10      80%  ⚠
Social posts published     5      30      17%
DMs sent                  43     100      43%

⚠ Campaigns sent is approaching the limit. Consider:
  • Upgrading to SCALE for 50 sends/month
  • Batching multiple promos into a single campaign
```

No write operations — purely read.
