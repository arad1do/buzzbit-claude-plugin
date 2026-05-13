---
name: buzzbit-popup-designer
description: Activates when designing or editing a popup — signup form, exit-intent offer, spin-the-wheel, cart-stickiness coupon, welcome bar. Use any time the user asks to create a popup or modify an existing one in BuzzBit X.
---

# BuzzBit X Popup Designer

A **popup** in BuzzBit X is an on-site overlay that shows up based on visitor behavior (about to leave, scrolled far, time elapsed, etc.) and collects an action — email signup, discount claim, survey response. Optionally connects to an email flow so the moment the customer submits, a welcome series starts.

## First, always

1. `get_workspace_brand` → primary color, accent color, fonts. The popup must feel like the rest of the site, not a generic template.
2. `list_popups` → what already exists. Two popups firing on the same trigger compete; check before adding another.
3. `list_popup_templates` *(when available — Phase 2)* → start from a proven pattern instead of from scratch.
4. If the popup should kick off an email flow on signup → confirm the flow exists and is `ACTIVE` (`list_flows`).

## Trigger types — 6 real ones (camelCase, exact)

The trigger value is set via the `showOn` field. Case-sensitive.

| Trigger | Fires when | Typical config |
|---|---|---|
| `pageLoad` | Page finishes loading | Use sparingly — most aggressive; reserve for high-intent pages |
| `exitIntent` | Mouse moves toward the browser chrome (desktop) or user scrolls fast upward (mobile) | The classic "wait, don't go" — best for first-time visitors |
| `scroll` | Visitor scrolls past a threshold (configured as `scrollPercentage`, 0–100) | 50% is the standard — they've read enough to care |
| `timeDelay` | Visitor has been on the page for N seconds (configured as `delaySeconds`) | 15–30 seconds for most cases — shorter feels pushy |
| `click` | Visitor clicks a specific element (configured as `clickSelector` CSS selector) | Use for "click to claim" CTAs embedded in page content |
| `intent` | Visitor signals high intent through scroll-back, hover patterns, or repeated visits | Most subtle — use for welcome offers to returning visitors |

## Targeting rules

Limit who sees the popup. All optional; combine freely.

| Field | Examples |
|---|---|
| Pages | `includePages: ["/products/*", "/collections/sale"]` / `excludePages: ["/cart", "/checkout"]` |
| Visitor type | `firstTimeOnly: true` / `returningOnly: true` / `loggedInOnly: false` |
| Device | `devices: ["desktop", "mobile", "tablet"]` |
| Geography | `countries: ["IL", "US"]` (ISO codes) |
| Frequency | `oncePerSession` (default) / `oncePerCustomer` / `everyDays: 7` |
| Referrer | `sources: ["instagram", "facebook"]` for ad-traffic-only popups |

Don't show a popup on `/cart` or `/checkout` unless explicitly asked — interrupting checkout is the fastest way to lose the sale.

## HTML constraints (subset of email rules)

The popup is rendered in the merchant's storefront, so it inherits site CSS but must not break it. Constraints:

- **No `<form>` wrappers.** Submission goes through the popup's built-in handler — wrapping in a form breaks it
- **Use button elements**, not anchor `<a>` styled as buttons — the click handler needs a real button
- **Inline styles only** — popups load before stylesheet rules in some themes; inline beats class
- **No `<script>` tags.** Sanitizer strips them; merchant gets a popup that does nothing
- **Max width 600px on desktop, 90vw on mobile** — wider feels like a takeover, narrower feels cramped
- **Single CTA** — one primary button per popup. Two competing CTAs cut conversion roughly in half

## Content shape

| Field | Purpose |
|---|---|
| `headline` | 4–8 words, leads with the value (`"15% off your first order"` not `"Sign up to our newsletter"`) |
| `subheadline` | Optional one-line elaboration |
| `body` | 1–2 sentences max; the popup is interruption, not an article |
| `cta.label` | Action verb (`"Claim my code"`, `"Get the discount"`) — never `"Submit"` |
| `cta.color` | From `brand.primaryColor` unless the popup is testing a contrasting CTA |
| `inputs` | The fields collected — email (required), firstName (optional), phone (optional) |
| `successMessage` | What shows after submit — keep it warm; never just "Thanks" |

## Connecting to a flow

If the popup should trigger an email flow on signup:

1. Confirm the flow exists and uses trigger `new_subscriber` (this is the only trigger that fires from popup submission — `cart_abandoned`/`order_placed`/etc. do NOT fire from popups)
2. Set `startFlowOnTrigger: true` and `flowId: <the flow's id>` on the popup
3. Verify the flow is `ACTIVE`, not `DRAFT` — popups linked to draft flows submit successfully but the flow never starts

The popup hands the customer's email + firstName (if collected) into the flow's variable context. The flow's emails can reference `{{customer.email}}` and `{{customer.firstName}}` immediately.

## A/B variant pattern

Two ways to test:

1. **Two popups, same trigger, different design.** Both fire (different visitors). Compare submission counts and downstream flow conversion. Lower-effort, faster signal.
2. **One popup with `variants: [{ ... }, { ... }]`** *(when supported by the popup service — check `list_popups` shape)* — proper split-test with traffic allocation. Use when sample sizes warrant it.

Always change exactly one variable per test (headline OR color OR CTA copy — not all three). Otherwise you can't tell what moved the needle.

## Standard build sequence

1. Gather context (the `First, always` block)
2. Decide trigger + targeting + frequency before composing copy. The wrong trigger sinks an otherwise-perfect popup.
3. Compose `headline` + `subheadline` + `body` + `cta` in product voice (`buzzbit-content-creator` skill helps)
4. Apply brand colors from `get_workspace_brand` to background, text, button
5. `create_popup_draft({ name, type, design, content, triggers, targeting, startFlowOnTrigger?, flowId? })` → popup saved as `DRAFT`
6. Surface the preview URL to the merchant. They review on the dashboard, see how it looks against the actual site
7. On their go-ahead: status flips to `ACTIVE` (via `update_popup_draft` setting `status: "ACTIVE"`)

## Anti-patterns

- **Don't activate without preview.** The popup is the most visible piece of UI on the site for new visitors.
- **Don't trigger on `pageLoad` for first-time visitors.** Aggressive popups kill bounce-rate.
- **Don't show a popup on every page.** Use `includePages` to limit to where the offer is relevant.
- **Don't collect more than email + firstName at first.** Every extra field cuts submission ~10%.
- **Don't promise a discount in the headline and bury the code in the success message.** Reveal the code immediately or send it via the connected flow's first email — but make it obvious which.
- **Don't reference an undefined trigger** (e.g. `"timeOnPage"` instead of `"timeDelay"`). The popup saves but never fires.

---

_Trigger names verified 2026-05-14 against `popupService.ts` `showOn` enum. The valid values are camelCase (`pageLoad`, `exitIntent`, `scroll`, `timeDelay`, `click`, `intent`) — older docs that used snake_case or different names were silently broken._
