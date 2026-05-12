---
name: buzzbit-billing-monitor
description: Inspect subscription status, invoices, and payment history. Loads when the user asks about billing, plan tier, invoices, charges, refunds, "how much am I paying", "am I near my limit", "did my card fail". Uses get_subscription_status, list_invoices, get_payment_history, get_workspace_limits.
---

# BuzzBit Billing Monitor

When the user wants to understand spend / billing / plan health, load this skill.

## When to load

- "What plan am I on?"
- "How much have I spent this year?"
- "Did my card fail recently?"
- "Show me my last 5 invoices"
- "Am I close to my plan limit?"
- "When does my subscription renew?"

## Workflow

### 1. The fast answer — subscription overview

```
get_subscription_status
```

Returns: `{ plan, status, amount, currency, currentPeriodStart, currentPeriodEnd, cancelAtPeriodEnd, lastFourDigits }`.

**Never** return the `cardcomToken`, raw card number, or any token-shaped string — the server already strips them, but don't paraphrase them into prose either.

### 2. "Am I near my limit?" — quota visibility

```
get_workspace_limits
```

Returns the merchant's plan tier + current-period quota counters
(`mcpApiCallsUsed`, `mcpDraftsCreated`, `mcpCampaignsSent`, `mcpSocialPostsPublished`, `mcpDmsSent`).

Display as `used / limit (used%)`. Flag any counter ≥ 80% as yellow, ≥ 95% as red. Concrete upgrade math: tell the merchant the next tier's limit on that specific counter, not a generic "upgrade for more."

### 3. Invoice history

```
list_invoices { limit?, status? }
```

`status` is one of `PENDING | COMPLETED | FAILED | REFUNDED`. To answer "what failed?", filter by FAILED and read the `failureReason` field.

### 4. Pattern analysis — payment history aggregate

```
get_payment_history { months: 12 }
```

Returns `{ window, counts, totals, lastFailure }`. Useful for:
- "How often did my card get declined?" → `counts.failedCount`
- "How much have I refunded?" → `totals.totalRefunded`
- "What's my net spend this year?" → `totals.net`

### 5. Combining with quota

If `get_payment_history.counts.failedCount > 0` AND the workspace plan is FREE/GROWTH, the merchant might be unintentionally throttled (subscription was active, payment failed, plan effectively downgraded). Surface this as a single insight: *"Your last payment failed; your account is currently on the X tier with limit Y. Update payment method to restore Z."*

## Privacy + safety

- Never return the `cardcomToken`, full card number, or any other raw payment identifier. The server strips them but don't try to derive them from anything else either.
- The `cardcomInvoiceUrl` on PaymentRecord rows is the merchant's downloadable PDF — that's safe to share.
- Don't speculate about what's wrong if `failureReason` is null. Just say "payment failed; reason not recorded."

## Anti-patterns

- **Don't auto-charge or change plan.** There is no MCP tool to upgrade / downgrade — that always happens in the dashboard. If the merchant asks, point them at `/settings/billing` and offer to summarize their current state.
- **Don't compare to other workspaces.** Cross-workspace data is invisible by design.
- **Don't read more than the merchant asked for.** Don't dump 12 months of invoices when they asked about today.

## Follow-ups to offer

- "Want me to email this summary to your finance team?" — `create_dm_draft`
- "Want to add this to a board for tracking?" — `create_board_item`
- "Want to set a flow that alerts you when quota hits 80%?" — `create_flow_draft` (the flow engine supports threshold triggers)
