---
name: buzzbit-overview
description: Master skill — always loads. Teaches Claude how to operate BuzzBit X via the MCP server. Trigger on any buzzbit / marketing automation / flows / campaigns / popups / segments task.
---

# BuzzBit X Operations — Overview

You are connected to the BuzzBit X marketing automation MCP server at `https://api.buzzbitx.com/mcp`. This skill teaches you how to use it effectively.

## What BuzzBit X is

BuzzBit X is a multi-channel marketing automation platform: email campaigns, email flows (drip sequences), social posts, DMs, popups, and customer segments. The merchant runs a Shopify store; BuzzBit X drives growth via lifecycle marketing.

The MCP server exposes ~48 tools across 11 domains. **Every tool is workspace-scoped** — the API key authenticates one workspace, and you see only that workspace's data. Cross-workspace access is impossible by design.

## Tool surface

**Read** (always free, no quota cost beyond fair-use API call counter):
- Analytics: `get_dashboard_overview`, `get_revenue_stats`, `get_growth_metrics`
- Customers: `list_customers`, `get_customer_details`, `search_customers`
- Orders: `list_orders`, `get_order_details`
- Products: `list_products`, `get_product_details`, `get_product_performance`
- Flows: `list_flows`, `get_flow_performance`
- Campaigns: `list_campaigns`, `get_campaign_metrics`
- Social: `list_social_posts`, `get_social_performance`
- Inbox: `list_conversations`, `get_conversation_messages`, `list_dm_templates`
- Popups: `list_popups`, `get_popup_performance`
- Segments + workspace: `list_segments`, `get_workspace_brand`, `get_workspace_limits`

**Write/Draft** (counts toward `mcpDraftsCreated`):
- `tag_customers`, `bulk_tag_customers`
- `create_segment`, `update_segment`
- `create_flow_draft`, `update_flow_draft`
- `create_campaign_draft`, `create_campaign_draft_with_html`, `update_campaign_draft`, `validate_email_html`, `bulk_create_email_campaigns`
- `create_social_post_draft`, `update_social_post_draft`, `bulk_create_social_posts`
- `create_popup_draft`, `update_popup_draft`
- `create_discount_code`
- `upload_media`

**Execute** (heavier counters — `mcpCampaignsSent`, `mcpSocialPostsPublished`):
- `send_campaign` — 30s undo window, returns `actionId`
- `activate_flow`
- `publish_social_post` — 30s undo window
- `delete_draft` — 24h soft-delete on supported types
- `cancel_pending_action` — abort a queued send/publish within 30s

## Operating principles

1. **Inspect before acting.** Before `send_campaign`, call `get_campaign_metrics` to see status. Before `tag_customers`, call `get_customer_details` to confirm identity. The merchant's data is real — destructive haste is bad.

2. **Drafts first, executes second.** Creating a draft is cheap (small quota). Executing is expensive (hard caps + undo windows). When asked to "send a welcome campaign", default to creating the draft and surfacing the `previewUrl` for human approval — only execute when explicitly asked.

3. **Quota awareness.** Before any bulk operation, call `get_workspace_limits` to check headroom. A 50-recipient bulk send on Growth tier (10 sends/month) might consume the entire monthly cap.

4. **Use the brand voice.** Before generating any customer-facing copy, call `get_workspace_brand`. Match the workspace's `brandVoice` (PROFESSIONAL / CASUAL / LUXURY / URGENT / FRIENDLY / FUNNY) and color palette.

5. **Validate HTML.** Before storing email HTML via `create_campaign_draft_with_html`, run it through `validate_email_html` first to surface warnings (e.g., disallowed tags) so you can adjust.

## Idempotency

Write tools support an optional `X-Idempotency-Key` header. If you retry a write within 24h with the same key, you get the cached response instead of duplicating the action. Use this for any "create N drafts in a loop" pattern.

## Error handling

Tool errors come back as `isError: true` with a structured `code` + `message` and sometimes extra fields:
- `QUOTA_EXCEEDED` — surface the upgrade URL to the merchant; don't silently retry
- `NOT_FOUND` — the resource doesn't exist or isn't in this workspace
- `INVALID_STATUS` — the resource exists but isn't in the right state for the action
- `CODE_ALREADY_EXISTS` — a discount code with this code is already taken

Always relay the error code and message to the merchant — don't paraphrase.
