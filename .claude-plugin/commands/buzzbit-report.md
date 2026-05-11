---
name: buzzbit-report
description: Compose a multi-section snapshot report for the connected BuzzBit X workspace (dashboard, revenue, growth, top products, recent campaigns).
---

# /buzzbit-report

Generates a weekly/monthly executive snapshot of the merchant's store.

## What I'll do

Run these MCP tool calls in parallel:

1. **`get_dashboard_overview`** — headline numbers (today's orders, revenue, AOV, active customers)
2. **`get_revenue_stats`** with `period: "last_7_days"` (or `last_30_days` if user says monthly)
3. **`get_growth_metrics`** with the same period — customer/order growth %
4. **`list_products limit: 10`** — top recent / inventory-relevant products
5. **`list_campaigns limit: 10`** — recent campaign performance
6. **`get_social_performance`** with the same period — engagement totals

Then render a single Markdown report grouped by section. Include:
- Period covered, generation timestamp
- Section headers (`## Overview`, `## Revenue`, …)
- For each section, the most important 3-5 numbers as bullets (not raw JSON dump)
- A "Notable" line per section calling out any spike, dip, or near-quota item

## Period selection

If the user says:
- "weekly report" / "this week" → `last_7_days`
- "monthly report" / "this month" → `last_30_days`
- "quarterly" → `last_90_days`
- Otherwise default to `last_7_days`

## Privacy

Do not include individual customer emails / phone numbers in the report — use aggregates only.

## Followups to offer

After delivering the report, propose:
- "Want me to email this to your team?" — would need `send_dm_template` or a `create_campaign_draft` aimed at the merchant's own admin email
- "Want me to save this as a campaign idea?" — `create_campaign_draft`
- "Want to dig into a specific segment?" — invoke `buzzbit-customer-analyzer` skill
