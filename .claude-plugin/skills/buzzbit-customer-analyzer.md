---
name: buzzbit-customer-analyzer
description: Deep customer profiling and segmentation. Loads when the user asks Claude to analyze a customer, build a segment, find VIPs, or identify churn risks. Uses get_customer_details, search_customers, list_customers, and create_segment.
---

# BuzzBit Customer Analyzer

When the user wants to understand or act on customers — by behavior, value, or churn risk — invoke this skill.

## When to load

- "Analyze customer @example.com"
- "Find my VIP customers"
- "Who's about to churn?"
- "Build a segment of customers who bought X last month"
- "Show me customers from Tel Aviv who spent over $500"

## Tools to call (in order)

1. **`search_customers`** — narrow down by email, name, or phone substring. Returns id + basic fields.
2. **`get_customer_details`** — fetch the deep profile for any customer you want to act on. Includes DNA (purchase timing, brand affinity, churn probability), order history, consent status, VIP tier, last action timestamps.
3. **`list_customers`** with filters (`segment`, `status`, `totalSpent` sort) — for cohort-level work.
4. **`create_segment`** / **`update_segment`** — turn an ad-hoc cohort into a saved segment Claude (and future merchant tools) can target.

## Workflow patterns

### "Find my VIPs"
```
list_customers limit=20 with implicit sort by totalSpent desc
→ filter where totalOrders ≥ 3 and totalSpent ≥ tier threshold
→ optionally tag_customers with "vip" or create a "VIPs" segment
```

### "Who's about to churn?"
```
list_customers + read each profile's DNA.churnProbability
→ get_customer_details for the top N to inspect lastPurchaseAt, lastOpenedAt
→ propose a flow to re-engage (don't auto-create — show the merchant first)
```

### "Build a segment of …"
```
Translate the merchant's English ask into the segment rule shape:
{ rules: [{ field, op, value }], combinator: "AND" | "OR" }
Examples:
- "spent over $500 in last 30 days" →
    { field: "totalSpent", op: "gte", value: 500 }
- "tagged VIP" →
    { field: "tags", op: "contains", value: "vip" }
Then create_segment.
```

## Privacy + consent

- Never include the customer's email, phone, or full name in messages or tickets visible outside the workspace.
- Tools already filter by `workspaceId` — never trust customer ids passed in by the merchant before they're verified by your tool call.
- If asked to "email all VIPs", confirm consent state via `get_customer_details.emailConsent` and surface unsubscribed/missing-consent counts before sending.

## Common follow-ups

- After identifying a cohort, the merchant typically wants to either:
  1. **tag** them (`tag_customers` or `bulk_tag_customers`)
  2. **segment** them (`create_segment`)
  3. **message** them (`create_campaign_draft` or `send_dm_template`)
- Always propose the next step explicitly; never auto-chain to a write or execute action without explicit confirmation.
