---
name: buzzbit-flow-builder
description: Activates when designing, editing, or activating an email flow — welcome series, cart recovery, post-purchase, birthday, back-in-stock, win-back, browse abandonment. Use for any task that builds a multi-step email automation in BuzzBit X.
---

# BuzzBit X Flow Builder

A **flow** in BuzzBit X is a directed graph of email steps with delay and branch logic, triggered when something happens for a customer — they signup, abandon a cart, place an order, have a birthday, and so on.

## First, always

Before composing nodes, gather context:

1. `get_workspace_brand` → primary color, accent color, font, voice. Reuse on every email so the flow feels on-brand.
2. `list_segments` → which audiences exist. If a flow targets one, name it explicitly.
3. `list_products` (when the flow references products — back-in-stock, price drop, post-purchase upsell) → top sellers or specific SKUs.
4. `get_workspace_limits` → confirm headroom in the `drafts` quota and the active-flow cap.
5. `list_flow_templates` *(when available — Phase 2)* → if a similar pattern already exists, clone and adapt rather than starting from scratch.
6. `list_template_variables(triggerType)` *(when available — Phase 2)* → confirm which variables the trigger emits before referencing them in copy.

## Triggers — the 10 most common (lowercase snake_case, always)

The trigger string is case-sensitive. The data layer accepts anything, but the engine only fires on these exact names. Wrong case or wrong word → silent failure.

| Trigger | Fires when | Variables available |
|---|---|---|
| `cart_abandoned` | A cart sits idle for the configured threshold (default 1h) | `cart.items[]`, `cart.total`, `cart.lastUpdatedAt`, `customer.firstName`, `customer.email` |
| `browse_abandoned` | A customer views products without adding to cart | `customer.firstName`, `customer.email`, `lastViewedProducts[]` |
| `customer_created` | New customer record appears (popup signup, Shopify customer create, manual add) | `customer.id`, `customer.email`, `customer.firstName`, `customer.lastName`, `customer.createdAt` |
| `new_subscriber` | Popup signup completed (specifically the popup→flow link) | `customer.email`, `customer.firstName`, `popup.name`, `popup.discountCode` |
| `order_placed` | Order created in the store | `order.id`, `order.total`, `order.currency`, `order.items[]`, `customer.firstName`, `customer.email` |
| `order_fulfilled` | Order marked fulfilled in Shopify | `order.id`, `order.trackingNumber`, `order.carrier`, `customer.firstName` |
| `refund_created` | Refund issued — listen-only, never trigger a refund from here | `order.id`, `refund.amount`, `customer.firstName` |
| `product_price_drop` | Product price reduced for a customer who viewed/wishlisted it | `product.id`, `product.name`, `product.oldPrice`, `product.newPrice`, `customer.firstName` |
| `product_back_in_stock` | Product stock > 0 for a customer who subscribed to the back-in-stock alert | `product.id`, `product.name`, `product.stock`, `customer.firstName` |
| `customer_birthday` | Cron job at 08:00 in the customer's timezone on their birthday | `customer.firstName`, `customer.email`, `customer.birthday` |

**See also**: 12 more triggers exist for advanced cases. They are not covered here in detail because most flows use one of the 10 above. Names: `checkout_abandoned`, `order_cancelled`, `order_paid`, `customer_updated`, `customer_inactive`, `product_low_inventory`, `fulfillment_created`, `instagram_comment`, `instagram_story_reply`, `instagram_follow`, `instagram_dm_received`, `SOCIAL_COMMENT`. Ask the merchant before using any of these — they have specific semantics worth confirming.

## Node types — exactly 5

| Type | Required fields | Optional fields |
|---|---|---|
| `trigger` | `eventType` (one of the names above) | `conditions` (filter — e.g. `{ "cart.total": { "gt": 100 } }` to only fire when cart > $100) |
| `delay` | `value` (number), `unit` (`seconds` \| `minutes` \| `hours` \| `days`) | none |
| `email` | `campaignId` (returned by `create_campaign_draft_with_html`) | none |
| `condition` | `field` (variable name, e.g. `customer.totalOrders`), `operator` (see below), `value` | `label` (human-readable, shown in canvas) |
| `goal` | `name` (e.g. `"purchased"`, `"opened"`) | `description` |

## Delay units — realistic ranges

| Unit | Sensible range | Don't go below |
|---|---|---|
| `seconds` | Almost never used — only for testing | 10 sec |
| `minutes` | 5–60 (immediate-feeling follow-up) | 1 min |
| `hours` | 1–48 (most cart/abandonment delays) | 1 hour |
| `days` | 1–30 (welcome series, birthday warm-up, win-back) | 1 day |

Don't use 5-second delays in customer-facing flows. The merchant looks lazy, customer feels spammed.

## Condition operators — what actually exists today

Two operator sets depending on where conditions live:

**Inside `condition` nodes** (the in-graph branching): **7 symbolic operators only**.

| Operator | Use |
|---|---|
| `>` `<` `>=` `<=` | Numeric comparisons (`customer.totalOrders >= 1`, `order.total > 100`) |
| `==` | Equality on strings/numbers/booleans |
| `!=` | Inequality |
| `contains` | Substring (on strings) or membership (on arrays) — `customer.tags contains "VIP"` |

**Inside `trigger.conditions`** (filtering when the trigger itself fires): **7 named operators**.

| Operator | Use |
|---|---|
| `gt` `gte` `lt` `lte` | Numeric (`{ "cart.total": { "gt": 100 } }`) |
| `contains` | Substring/array membership |
| `in` | Value in a list — `{ "customer.country": { "in": ["IL","US"] } }` |
| `keywords` | Match against a keyword set (for chat/DM triggers) |

Operators not yet supported: `startsWith`, `endsWith`, `regex`, `between`, date math (`before`, `after`, `withinDays`, `olderThanDays`), array-length (`lengthGt`, `lengthLt`). If the merchant asks for one of these, propose a workaround using the operators that exist — for example, "this week's birthdays" → use the existing `customer_birthday` trigger rather than a date-comparison condition.

## Graph integrity rules

Before submitting to `create_flow_draft`, the graph must satisfy:

1. **Single trigger** — exactly one `trigger` node, and it has no incoming edges
2. **At least one goal** — every path from the trigger reaches a `goal` node (or the merchant's explicit equivalent end-state)
3. **No orphans** — every non-trigger node has an incoming edge
4. **No cycles** — a customer must never loop back to an earlier node
5. **Every `condition` node has both branches** — `true` and `false` outgoing edges, both reachable
6. **Every `email` node references a real `campaignId`** — create the draft first, copy the returned id
7. **Every `delay` has `value` + `unit`** — no zero, no missing unit

Call `validate_flow_graph` *(Phase 2)* before `create_flow_draft`. Fix every error before submitting.

## Variable resolution at runtime

`{{customer.firstName}}` in an email body resolves against the trigger's payload at send time. If the variable doesn't exist in that trigger's context, the recipient sees a literal empty string (not the placeholder). Two implications:

- Don't reference `{{cart.total}}` in a flow triggered by `customer_birthday` — there's no cart context
- Use `list_template_variables(triggerType)` *(Phase 2)* to confirm available variables before composing
- The campaign draft preview (`validate_email_html` warnings) catches missing-variable issues — re-validate after editing

## Standard build sequence

1. Gather context (the `First, always` block)
2. Sketch the flow in plain English — what triggers, how many emails, what delays, what condition splits, what goals
3. Create discount codes if needed (`create_discount_code`) and save the ids — flows can reference them in email bodies
4. Compose each email's HTML (use `buzzbit-email-designer` skill) → `validate_email_html` → fix warnings → `create_campaign_draft_with_html` → save the draft id
5. Compose the graph: nodes (with the campaign ids embedded in email-type nodes) + edges
6. `validate_flow_graph` *(Phase 2)* → fix every error
7. `create_flow_draft({ name, description, trigger, nodes, definition })` → flow created in `DRAFT` status
8. Surface the preview URL to the merchant — **don't auto-activate**. The merchant reviews in the dashboard and decides when to flip to ACTIVE
9. On their go-ahead: `activate_flow(flowId)` (30-second undo window applies)

## Anti-patterns

- **Don't activate without preview.** Even on simple flows, the merchant should eyeball the canvas + email content before going live.
- **Don't reuse discount codes across flows** unless explicitly asked. Each flow's codes should be unique so attribution stays clean.
- **Don't generate emails longer than 600 words.** Customers don't read past the first scroll.
- **Don't use raw HTML without `validate_email_html` first.** It catches missing unsubscribe links and auto-injects the footer.
- **Don't compose with uppercase trigger names** (e.g. `CART_ABANDONED`). The engine listens for lowercase; the draft saves successfully but the flow never fires.
- **Don't reference variables that aren't in the trigger context** — the email will render blank.

---

_Trigger names verified 2026-05-14 against `FlowTriggerType` in the flow engine. Prior versions of this skill used uppercase names that the engine silently ignored. If you see uppercase names in older docs or examples, treat them as incorrect._
