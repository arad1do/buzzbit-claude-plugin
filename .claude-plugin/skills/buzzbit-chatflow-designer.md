---
name: buzzbit-chatflow-designer
description: Activates when designing or editing a chat flow — WhatsApp / Instagram / Messenger DM automation. Use any time the user asks to build a bot that replies to customer messages, qualifies leads through chat, handles FAQ via DM, or automates order-status replies.
---

# BuzzBit X ChatFlow Designer

A **chat flow** is a tree of question-and-response nodes that activates when a customer messages the business on WhatsApp, Instagram, or Messenger. Different from an email flow: chat flows are interactive, branch on what the customer types, and have to handle the unexpected reply gracefully.

## First, always

1. `get_workspace_brand` → tone of voice. A bot that sounds nothing like the brand erodes trust faster than no bot at all.
2. `list_chatflows` → what's already automated. Two flows triggered on `whatsapp_first_message` will both fire; check before adding.
3. `list_chatflow_templates` *(when available — Phase 2)* → 6 prebuilt patterns covering 80% of real use cases.
4. Confirm the messaging channel the merchant wants is connected (`list_integrations health` → WhatsApp / Instagram / Messenger status).

## Trigger types — what really fires the flow

Lowercase snake_case, case-sensitive. Verified against `chatFlowExecutor`.

| Trigger | Fires when | Channels |
|---|---|---|
| `whatsapp_first_message` | Customer messages the business on WhatsApp for the first time ever | WhatsApp |
| `whatsapp_message` | Any incoming WhatsApp message after the first | WhatsApp |
| `whatsapp_keyword` | A WhatsApp message matches a configured keyword set | WhatsApp |
| `instagram_dm_received` | Direct message arrives on Instagram | Instagram |
| `instagram_comment` | Comment on an Instagram post matches rules | Instagram |
| `instagram_story_reply` | Reply to an Instagram story | Instagram |
| `keyword_match` | Generic keyword match across enabled channels | Any |

The trigger config can also include a keyword list (`keywords: ["order", "status", "where is"]`) — only messages containing one of those words fire the flow.

## The WhatsApp 24-hour rule (product fact, not infrastructure)

WhatsApp Business Platform allows the business to message a customer freely for **24 hours after the customer's last incoming message**. Outside that window, only **pre-approved template messages** can be sent. Practical implications for chat flows:

- A first-message flow can send any messages within 24h of the customer's opening message
- A follow-up flow that fires 26 hours later must use a `whatsapp_template` node, not a regular `message` node
- The session window resets every time the customer messages back — keep the conversation moving

Instagram and Messenger have similar windows (24 hours then a relaxed 7-day rule for Instagram). The chat engine enforces these automatically — your flow doesn't have to track the clock, but if you compose a `message` node outside-window, it fails to send.

## Node types — 8 real ones

| Type | Purpose | Required fields |
|---|---|---|
| `trigger` | Entry point — exactly one per flow | `triggerType` (one of the names above), optional `keywords` |
| `message` | Send a text reply | `content` (the text) |
| `whatsapp_template` | Send an approved WhatsApp template (use outside the 24h window or for promotional sends) | `templateName`, `templateVars` |
| `delay` | Wait before the next node | `value`, `unit` (`seconds` \| `minutes` \| `hours`) |
| `condition` | Branch on customer reply, tags, profile data | `field`, `operator`, `value` (same 7 symbolic operators as email flows) |
| `ai_response` | Generate a contextual reply using the workspace's AI agent | `prompt`, optional `tone` |
| `handoff` | Pass the conversation to a human (inbox queue) | `assignTo` (optional team member id), `reason` |
| `end` | Explicit termination | none — used to make the flow's intent visible |

## The default-path rule

The single biggest mistake in chat flows: not handling unexpected replies. Customers say weird things. Every `condition` node must have a default branch.

**Required pattern:**

1. `condition` node checks for expected reply (`message contains "yes"` or `message contains "order"`)
2. True branch: continue the conversation
3. False branch: send a clarifying message ("Sorry, I didn't catch that — try replying with `yes` or `no`")
4. After two consecutive unrecognized replies on the same `condition` → route to `handoff`

Without this, the bot loops forever on a misunderstood reply and the customer rage-uninstalls or leaves a 1-star review.

## Patterns — 4 that cover most real use cases

| Pattern | Trigger | Sequence |
|---|---|---|
| Order status | `whatsapp_keyword` with keywords `["order", "status", "where", "tracking"]` | `message` "Order number?" → `condition` (extract numbers) → if found: API lookup → `message` with status; if not found: `handoff` |
| FAQ tree | `whatsapp_first_message` or `instagram_dm_received` | `message` "Hi! What can I help with?" with quick replies → `condition` per reply → leaf `message` per FAQ → final `handoff` for "still need help" |
| Lead qualifier | `instagram_dm_received` | Ask 3 questions via `message` + `condition` chains (budget, timeline, fit) → if qualified: `handoff` to sales; if not: `message` "Here's our self-serve link" + `end` |
| Abandoned-checkout-from-DM | `whatsapp_keyword` with `["cart", "checkout", "buy"]` | `message` "Want me to send your cart back?" → `condition` "yes" → API pull → `message` with the cart link → `delay 24h` → `whatsapp_template` reminder |

## Standard build sequence

1. Gather context (the `First, always` block)
2. Sketch the conversation in plain English — what the customer says, what the bot replies, what branches matter, where a human takes over
3. Identify every `condition` branch and write its default. **Don't compose until each `condition` has true / false / default-after-2-misses.**
4. For WhatsApp flows that may extend past 24 hours: pre-approve the template messages in the dashboard (templates require Meta review, can take 24h to approve)
5. `create_chatflow_draft({ name, description, trigger, nodes, definition })` *(Phase 2 tool)* → flow saved as `DRAFT`
6. Surface preview URL — merchant tests by sending a message to the connected number/account themselves
7. On their go-ahead: `activate_chatflow(id)` *(Phase 2)* — flow goes live, 30-second undo applies

## Anti-patterns

- **Don't use a `message` node for promotional content after the 24h window.** It will silently fail to send.
- **Don't ask more than 3 questions before a handoff option.** Customers patience for chatbots peaks around question 2.
- **Don't reference variables that aren't in the trigger payload** — e.g. `{{customer.firstName}}` won't resolve on a first-message flow if the customer hasn't been identified yet.
- **Don't activate a flow that hits a `whatsapp_template` node without the template being pre-approved.** Confirm `templateName` exists and has status `APPROVED`.
- **Don't loop a `condition` back to its own input.** Two misses → `handoff`. Always.
- **Don't compose with PRD-fabricated node types** like `entry`, `question`, `branch`, `tag` — they don't exist in the engine; the flow saves but won't execute past the unknown node.

---

_Node and trigger names verified 2026-05-14 against `chatFlowExecutor.ts`. The valid node types are `trigger`, `message`, `delay`, `condition`, `ai_response`, `whatsapp_template`, `handoff`, `end` — older specs that used `entry`/`question`/`branch`/`tag` were aspirational and never shipped to the engine._
