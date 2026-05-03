---
name: buzzbit-flow-builder
description: Activates when creating, editing, or activating an email flow — welcome series, cart abandonment, post-purchase, win-back, etc. Trigger on any task involving multi-email drip sequences in BuzzBit X.
---

# BuzzBit X Flow Builder

A "flow" in BuzzBit X is a directed graph of email steps with delay/branch logic, triggered by a customer event (CART_ABANDONED, ORDER_PLACED, CUSTOMER_CREATED, POPUP_SUBMIT, etc.).

## Standard build sequence

1. **Inspect the workspace.**
   - `get_workspace_brand` → primary color, voice
   - `list_segments` → which audiences exist (or use `create_segment` first)
   - `list_products` (if the flow references products) → top sellers
   - `get_workspace_limits` → confirm headroom in `drafts` quota and `active flows` cap

2. **Plan the flow before writing nodes.** Cart abandonment example:
   - Trigger: `CART_ABANDONED`
   - Step 1: Email at +1h, no discount, "Did you forget something?"
   - Step 2: Email at +24h, 10% off code, "Here's a little incentive"
   - Step 3: Email at +3d, social proof, "What others are saying"
   - Step 4: Email at +7d, 15% off final code, "Last chance"
   - Total 4 emails, 2 discount codes (CART10, CART15)

3. **Create the discount codes first.**
   ```
   create_discount_code({code: "CART10", type: "PERCENTAGE", value: 10, expiresAt: <30 days>})
   create_discount_code({code: "CART15", type: "PERCENTAGE", value: 15, expiresAt: <30 days>})
   ```
   Save the returned IDs — you'll reference them in email content.

4. **Generate the email HTML in your own context.** Use the brand voice from step 1. Then for each:
   ```
   validate_email_html({html: <your html>}) → review warnings, adjust
   ```

5. **Create the flow as a DRAFT.**
   ```
   create_flow_draft({
     name: "Cart Abandonment",
     description: "4-step recovery series",
     trigger: "CART_ABANDONED",
     nodes: <ReactFlow JSON with email steps + delay nodes>
   })
   ```

6. **Surface the preview URL to the merchant — don't auto-activate.** The merchant reviews in `/flows/{id}` and decides when to flip to ACTIVE. If they say "go ahead and activate":
   ```
   activate_flow({flowId})
   ```

## Nodes JSON shape

Flows use ReactFlow under the hood. A minimal node graph:
```json
{
  "nodes": [
    {"id": "trigger", "type": "trigger", "data": {"eventType": "CART_ABANDONED"}},
    {"id": "delay-1", "type": "delay", "data": {"hours": 1}},
    {"id": "email-1", "type": "email", "data": {"campaignId": "<draft id>"}}
  ],
  "edges": [
    {"source": "trigger", "target": "delay-1"},
    {"source": "delay-1", "target": "email-1"}
  ]
}
```

The merchant's UI accepts more node types (branch, condition, segment-update, etc.). Keep it simple — fewer nodes is more maintainable.

## Anti-patterns

- **Don't activate without preview.** Even on simple flows, the merchant should eyeball the graph + email content before going live.
- **Don't reuse discount codes across flows** unless explicitly asked. Each flow's codes should be unique so attribution is clean.
- **Don't generate emails longer than 600 words.** Real customers don't read past the first scroll.
- **Don't use raw HTML without `validate_email_html` first.** The sanitizer strips dangerous tags but also catches things like missing unsubscribe links — let it auto-inject the footer.
