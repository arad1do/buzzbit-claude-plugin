# BuzzBit X — Plugin Marketplace Submission

Submission-ready metadata for the Anthropic Claude Code marketplace.
Copy/paste into whatever form Anthropic provides when the directory opens.

---

## Plugin name

**BuzzBit X**

## Tagline (≤80 chars)

E-commerce marketing automation — flows, campaigns, social, segments — in your terminal.

## Short description (≤200 chars)

Connect Claude Code to BuzzBit X. Read store data, draft email flows, schedule social posts, manage segments, and run analytics — without leaving the terminal. 48 MCP tools, 4 skills, 3 commands.

## Long description (marketplace listing body)

BuzzBit X is a multi-tenant marketing automation SaaS for Shopify, WooCommerce, and Wix stores. This plugin connects Claude Code to a merchant's BuzzBit X workspace over the Model Context Protocol, surfacing the platform's ~48 tools as native Claude capabilities.

### What you can do from Claude Code

- **Read store data**: customers, orders, products, campaigns, flows, popups, segments, conversations — all scoped to your workspace.
- **Draft and edit content**: email campaigns (text or HTML), social posts, popups, DM templates, flows. Sanitized server-side before save.
- **Send safely**: send a campaign or publish a social post with a 30-second undo window. Cancel mid-flight with `cancel_pending_action`.
- **Tag and segment**: bulk-tag customers, build segments from JSON rule definitions, target them in subsequent sends.
- **Analyze**: revenue, growth, top products, campaign metrics, flow performance, social engagement — across `last_7_days` / `last_30_days` / `last_90_days` periods.

### What's bundled

**4 auto-loading skills:**
- `buzzbit-overview` — master skill, always loads
- `buzzbit-flow-builder` — creating/editing email flows
- `buzzbit-content-creator` — generating email/social/popup copy
- `buzzbit-customer-analyzer` — deep profiling and segmentation

**3 slash commands:**
- `/buzzbit-status` — tier, quota usage, last-7-days metrics
- `/buzzbit-quota` — detailed quota usage with near-limit warnings
- `/buzzbit-report` — multi-section markdown executive snapshot

### Authentication

Bearer token (`bz_live_…` API key generated at `https://buzzbitx.com/settings/integrations` → Claude card). Workspace isolation is enforced server-side on every tool call.

### Pricing

The plugin itself is free. MCP access requires a BuzzBit X subscription on the **Growth** plan or higher.

| Tier | API calls/mo | Drafts | Sends | Social | DMs |
|---|---|---|---|---|---|
| Growth | 5,000 | 100 | 10 | 30 | 100 |
| Scale | 25,000 | 500 | 50 | 200 | 1,000 |
| Agency | 100,000 | unlimited | 500 | 1,000 | 10,000 |
| Enterprise | custom | custom | custom | custom | custom |

### Privacy + safety

- Every tool call writes an audit log (workspace, key, tool, duration, outcome).
- Send/publish tools go through a 30-second undo queue, server-side, before they take effect.
- API keys are bcrypt-hashed at rest. Revoke at any time from Settings → Integrations.
- Per-merchant tool gating: merchants can disable individual tools or whole categories from the Claude card without revoking the key.

---

## Repository

`https://github.com/arad1do/buzzbit-claude-plugin`

## Latest release

`v1.1.0` — `https://github.com/arad1do/buzzbit-claude-plugin/releases/tag/v1.1.0`

## Homepage

`https://buzzbitx.com/claude`

## Docs

`https://buzzbitx.com/docs/claude`

## Categories

- Marketing
- E-commerce
- Analytics
- Workflow automation

## Author

- Name: BuzzBit X
- Email: arad1doron@gmail.com
- Website: https://buzzbitx.com

## License

MIT

## Screenshots / demo

(Attach when uploading.)

1. Settings → Integrations → Claude card → 5-tab control panel (Keys / Tools / Activity / Usage / Setup).
2. 3-step Connect Claude wizard (first-time UX).
3. Claude Code session running `/buzzbit-report`, showing the rendered markdown.
4. Claude Code session running `buzzbit-customer-analyzer` skill — analyzing a VIP cohort.

## Test workspace

For Anthropic review: contact arad1doron@gmail.com for a temporary test workspace + API key.

---

## Pre-submission checklist

- [x] Manifest schema validates (4 skills, 3 commands, 1 MCP server)
- [x] All skills + commands load without errors against the live MCP server
- [x] LICENSE present (MIT)
- [x] README explains install, usage, troubleshooting
- [x] GitHub repository is public
- [x] Tag `v1.1.0` exists
- [x] Homepage and docs links resolve
- [ ] Anthropic-required screenshots captured (do when submitting)
- [ ] Marketplace listing form filled in (do when Anthropic opens submission)
