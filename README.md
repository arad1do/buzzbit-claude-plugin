# BuzzBit X — Claude Code Plugin

Connect Claude Code to your [BuzzBit X](https://buzzbitx.com) workspace. Build flows, draft campaigns, schedule social posts, manage segments, and run analytics — without leaving your terminal.

## Install

```bash
/plugin marketplace add arad1do/buzzbit-claude-plugin
/plugin install buzzbit
```

When prompted, paste your BuzzBit X API key:

```
Enter your BuzzBit API key: bz_live_...
```

You can generate one at **Settings → Integrations & API → Claude** in the BuzzBit X dashboard.

## What you get

- **MCP server connection** to `https://api.buzzbitx.com/mcp` — **~102 tools** across analytics, customers, orders, products, flows, campaigns, social posts, popups, segments, discounts, media, broadcasts, boards, content, support, team / RBAC, billing, integrations, webhooks, and workspace metadata.
- **7 skills** that auto-load when relevant:
  - `buzzbit-overview` — master skill, always loads
  - `buzzbit-flow-builder` — for creating/editing email flows
  - `buzzbit-content-creator` — for generating email/social/popup copy
  - `buzzbit-customer-analyzer` — for deep customer profiling and segmentation
  - `buzzbit-email-designer` — for email design with brand grounding + sanitizer awareness
  - `buzzbit-team-manager` — for inviting / role-changing / removing workspace members (RBAC-aware)
  - `buzzbit-billing-monitor` — for subscription / quota / invoice / payment-history questions
- **3 slash commands**:
  - `/buzzbit-status` — quick view of tier, quota usage, and last-7-days metrics
  - `/buzzbit-quota` — detailed quota usage with warnings for near-limit counters
  - `/buzzbit-report` — multi-section markdown executive snapshot

## Usage examples

**Workflow audit:**
```
> What's my best-performing campaign this month?
```

**Build a flow:**
```
> Create a cart abandonment flow with 4 emails: 1h reminder,
  24h with 10% off, 3d social proof, 7d final 15% off.
```

**Bulk content:**
```
> Create 10 Instagram posts for my summer collection.
  One per day starting Monday. Use my best-selling products.
```

**Status check:**
```
> /buzzbit-status
```

## Quota tiers

The MCP server enforces per-tier monthly limits. View live usage with `/buzzbit-status` or the `get_workspace_limits` tool.

| Tier | API calls | Drafts | Sends | Social posts | DMs |
|---|---|---|---|---|---|
| Growth | 5,000 | 100 | 10 | 30 | 100 |
| Scale | 25,000 | 500 | 50 | 200 | 1,000 |
| Agency | 100,000 | unlimited | 500 | 1,000 | 10,000 |
| Enterprise | custom | custom | custom | custom | custom |

Upgrade at https://buzzbitx.com/settings/billing.

## Safety

- **Workspace isolation** is enforced at the MCP server. Your API key sees only your workspace's data — no cross-tenant access is possible.
- **30-second undo window** on `send_campaign` and `publish_social_post`. Use `cancel_pending_action` to abort.
- **API keys** are bcrypt-hashed at rest. Revoke any time at Settings → Integrations & API.
- **Audit logs** record every tool call with workspace, key, tool name, duration, and outcome.

## Troubleshooting

- **`MCP not available on your plan`** — your workspace is on FREE tier. Upgrade to GROWTH or higher.
- **`QUOTA_EXCEEDED`** — you've hit a monthly counter. Wait for the period reset or upgrade.
- **`Invalid or expired API key`** — generate a new key in Settings → Integrations & API.
- **`Rate limit exceeded`** (HTTP 429) — slow down or upgrade your tier.

## Development

This repo contains:
- `.claude-plugin/plugin.json` — manifest
- `.claude-plugin/skills/` — auto-loading skills
- `.claude-plugin/commands/` — slash commands

To iterate locally, edit the markdown / JSON and reinstall via `/plugin reinstall buzzbit`.

## License

MIT — see [LICENSE](LICENSE).
