---
name: buzzbit-cross-channel-orchestrator
description: Activates when the merchant wants to promote something across multiple channels at once — "announce the new collection on email and Instagram", "send the Black Friday promo through email + popup + SMS", "share this offer everywhere". Use when a single brief should produce coordinated drafts across email, social, popup, SMS, and DM.
---

# BuzzBit X Cross-Channel Orchestrator

When the merchant says *"promote X"*, *"announce Y everywhere"*, or *"send this through email and Instagram"*, they don't want to repeat the same brief five times. They want **one prompt → coordinated drafts on every relevant channel, all on-brand, all consistent**. That's what this skill does.

## First, always

1. `get_workspace_brand` → primary color, accent color, font, voice. Every channel inherits these so the campaign feels like one promotion, not five separate ones.
2. `list_segments` → if the merchant names an audience, find its id. Cross-channel campaigns can target the same segment across email + DM + SMS.
3. `list_products` (when promoting specific items) → confirm SKUs, prices, stock. If a product is out of stock on the social post but in stock on the email, the campaign embarrasses the brand.
4. `list_integrations health` → confirm each channel the merchant wants is connected (email domain verified, Instagram linked, WhatsApp Business approved, etc.). Don't compose a draft for a disconnected channel — it can't go live anyway.

## Channel selection — when each one fits

Don't propose every channel for every brief. The wrong mix dilutes the message and burns audience trust.

| Channel | Best for | Avoid when |
|---|---|---|
| **Email** | The default. Detailed offer, products grid, longer copy, clear CTA. Works for promotions, launches, news. | Time-critical (people check email hours later, not minutes). |
| **Popup** | Catching site visitors during the promo window. Pairs perfectly with an email — popup converts the email-clicker into a sale once they're on the site. | Pre-purchase signup flows; not a fit for one-off promo brief unless the brief includes a site-visit phase. |
| **Social post** | Brand-building, reach beyond owned audience, time-relative announcements. Lives on the profile for weeks. | Personal / private offers (deals limited to a segment shouldn't be public). |
| **DM template** | Direct, conversational, intimate. Best for re-engagement of customers who already messaged the business or for high-value targets. | Cold outreach — the 24h WhatsApp window means you can only DM customers who messaged recently or who have opted in. |
| **SMS** | Urgent, time-boxed offers (`"sale ends in 4 hours"`). Highest open rate, lowest tolerance for noise. | Anything that can wait 12 hours. Cost per send adds up. |

## Decision matrix — propose, then confirm

When the merchant says *"announce the spring collection"*, don't dive into composition. **Propose the mix, ask once, then build.**

| Brief shape | Propose |
|---|---|
| Product launch / collection drop | Email + Instagram post + popup (on collection page) |
| Time-boxed sale (24–72h) | Email + SMS (final hours) + popup + Instagram story |
| Re-engagement / win-back | Email + DM template (for already-engaged customers only) |
| Brand storytelling / news | Email + Instagram post + blog (no popup, no SMS) |
| VIP-only offer | Email + DM template (don't post publicly — undermines the exclusivity) |

Present the proposal in one sentence: *"I'd suggest email + popup + Instagram for this — email goes Friday morning, popup activates on the collection page, Instagram post Friday afternoon. Confirm?"*

## Timing coordination — don't fire everything at once

Audience overlap fatigue is real. The same customer might see an email, a popup, and an Instagram story in 20 minutes. By the third, they tune out.

| Channel | When to fire (relative to email send time T=0) |
|---|---|
| Popup | T − 12 hours (active before the email, catches early site visitors) |
| Email | T (the announcement) |
| Instagram post | T + 4 hours (gives the email first crack, reinforces afterward) |
| Instagram story | T + 6 hours |
| SMS (final hours) | T + 36 hours, only for time-boxed sales |
| DM template | Same window as SMS, only for opted-in / recent-conversation contacts |

These are defaults — adjust per timezone, audience, and offer urgency.

## Consistency rules

Across every channel: **same headline, same CTA, same colors. Body adapts per channel.**

| Element | Email | Popup | Social | SMS | DM |
|---|---|---|---|---|---|
| Headline | Full subject line | Same headline, possibly trimmed | Same headline as caption opener | Trimmed to ≤40 chars | Same opener |
| Body | 80–200 words | 1–2 sentences | Caption 50–120 words | ≤140 chars | 30–60 chars |
| CTA | Button with link | Button with link | Bio link / story sticker | Trackable short link | Inline link |
| Visual | Brand colors, hero image | Brand colors, single CTA | Brand colors, square crop | None | Inline emoji optional |
| Personalization | `{{customer.firstName}}` | `{{customer.firstName}}` if known | Generic | `{{customer.firstName}}` if known | `{{customer.firstName}}` |

Don't change the offer mid-campaign. If the email says "25% off" and the Instagram caption says "30% off", the smart customer takes the better one and the merchant loses margin.

## Single-prompt expansion

The merchant brief is usually one sentence: *"Promote the new linen collection, 20% off this weekend."* Internal expansion before composing:

1. **Audience** — everyone? a segment? all subscribers? VIP only? Ask if unclear.
2. **Channels** — propose mix from the matrix above. Confirm.
3. **Offer details** — discount code? Free shipping threshold? End time?
4. **Visual asset** — does the merchant have product images? Should I generate from product catalog?
5. **Tone** — match brand voice from `get_workspace_brand`.

Then call `compose_cross_channel_campaign({ brief, channels, scheduledFor, segmentId? })` *(Phase 2 tool)* with the merged context. The tool internally calls the per-channel draft tools (`create_campaign_draft_with_html`, `create_popup_draft`, `create_post_draft`, etc.) and returns a bundle of draft ids.

## Standard build sequence

1. Gather context (the `First, always` block)
2. Propose the channel mix + timing schedule in one sentence. Wait for confirmation.
3. Ask for any missing context (audience segment, end time, discount details)
4. `compose_cross_channel_campaign(...)` → returns `{ campaignDraftId?, popupDraftId?, socialDraftId?, smsDraftId?, dmDraftId? }` with preview URLs
5. Surface each preview URL to the merchant in one summary block: *"Email: <url> · Popup: <url> · Instagram: <url>"*
6. On their per-channel approval → activate via the channel-specific tool (`send_campaign`, popup `update_popup_draft` to `ACTIVE`, `publish_post`, etc.)

Quota note: each channel counted is one draft against the workspace's `drafts` quota. A 5-channel campaign costs 5 draft slots.

## Anti-patterns

- **Don't compose for every channel.** Five drafts for a brief that needs two is wasteful and dilutes the campaign.
- **Don't fire all channels in the same minute.** Audience fatigue. Use the timing table.
- **Don't mismatch offers across channels.** Customer sees discrepancy → takes the better one OR distrusts the brand.
- **Don't activate all drafts at once.** The merchant reviews each preview before going live — let them approve per channel.
- **Don't propose SMS without confirming the merchant has SMS quota and customer opt-in.** Cold SMS is illegal in most regions.
- **Don't propose DM templates outside of WhatsApp's 24h session window** — most contacts won't be eligible; the campaign reaches a tiny fraction of the intended audience.

## When this skill should NOT activate

- The merchant explicitly names a single channel ("just send the email") — skip orchestration, hand off to `buzzbit-email-designer`.
- The merchant is editing an existing campaign / flow / popup — different concern; hand off to the channel-specific skill.
- The merchant asks "what should I send next?" without a brief — that's a strategy question; surface options, don't compose.

---

_Channel timing and decision matrix are recommendations distilled from typical SaaS e-commerce patterns. The merchant always has the final call._
