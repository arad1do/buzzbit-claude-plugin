---
name: buzzbit-email-designer
description: Design beautiful, deliverable email campaigns. Loads when the user asks Claude to design, draft, build, or compose an email campaign / newsletter / promo blast. Knows the server-side sanitizer's whitelist, the workspace's brand, mobile-first table syntax, and the 102 KB Gmail-clipping threshold.
---

# BuzzBit Email Designer

When the user wants to design a campaign — promo, newsletter, abandonment recovery, win-back — load this skill. It teaches Claude what good email HTML looks like *for this specific sanitizer* and how to ground each design in the merchant's brand.

## When to load

- "Design a Black Friday email"
- "Write a welcome series for new subscribers"
- "Build a cart abandonment email with 10% off"
- "Make me a newsletter from my last 5 blog posts"
- "Compose a back-in-stock notification"

## Workflow

### 1. Always start with brand + context

Before writing a single tag of HTML, fetch:

```
get_workspace_brand       → primaryColor, secondaryColor, fontFamily, brandVoice,
                            logoUrl, companyName, footerHtml, senderName
list_email_templates      → existing templates you can clone-and-adapt
                            (filter by category for fastest match)
```

If the merchant asks "use my brand colors", grounding starts here.

### 2. Pick a layout — six patterns that always work

The server's `sanitize-html` whitelist allows these tags: `html, body, table, tr, td, th, div, span, p, a, img, h1-h6, ul, ol, li, br, hr, strong, em, b, i, u, blockquote, code, pre, font, center`. **Tables are mandatory** for cross-client compatibility (Outlook still relies on Word's table engine; flexbox/grid are stripped or ignored).

Six patterns:

1. **Hero + CTA** — full-width image, headline, body, button
2. **Two-column** — hero on left, copy on right (or stack on mobile)
3. **Three-product grid** — 3-up product cards with image / name / price / "Shop"
4. **Editorial / newsletter** — multiple stacked content blocks with section dividers
5. **Plain-text-like** — minimal HTML, no images, high deliverability
6. **Receipt / transactional** — table-driven order summary

Always end with: brand-colored CTA → footer (unsubscribe + business address auto-injected by the server).

### 3. Mobile-first responsive pattern

The only responsive technique that works across Gmail mobile + Outlook is **percentage-width tables with stacked columns via media queries embedded in `<style>`**:

```html
<table role="presentation" width="100%" cellspacing="0" cellpadding="0">
  <tr>
    <td align="center" style="padding: 20px;">
      <table role="presentation" width="100%" style="max-width: 600px;">
        <!-- single column at <600px -->
        <tr>
          <td class="stack" width="50%" valign="top" style="padding: 10px;">
            …left col…
          </td>
          <td class="stack" width="50%" valign="top" style="padding: 10px;">
            …right col…
          </td>
        </tr>
      </table>
    </td>
  </tr>
</table>
<style>
  @media only screen and (max-width: 600px) {
    .stack { display: block !important; width: 100% !important; }
  }
</style>
```

### 4. Subject + preheader rules

- **Subject** ≤ 50 chars (most clients truncate after that on mobile).
- **Preheader** ≤ 100 chars — it appears in the inbox preview line after the subject.
- Emoji: at most 1, at the start. Never punctuation-spam.
- Avoid spam triggers: ALL CAPS, "FREE!!!", "$$$", "RE:". The sanitizer doesn't strip these, but the deliverability score drops.

### 5. The 102 KB Gmail-clipping threshold

Gmail clips emails > 102 KB and hides the unsubscribe footer behind a "View entire message" link, which hurts both deliverability and compliance. Stay under **80 KB** for safety.

The sanitizer returns `estimatedSizeKb` in its response — check it. If over, suggest:
- Use fewer/smaller images (host them, link via `<img src="">`, don't inline)
- Drop redundant `<style>` rules
- Split a long newsletter into a short email + "Read more on the blog" link

### 6. Variable substitution

Use `{{firstName}}`, `{{customerEmail}}`, `{{productName}}`, etc. The sanitizer's `detectedVariables` array confirms what was found — show this to the merchant so they know what dynamic fields are required.

### 7. Persistence

Two paths:

- **One-off campaign**: `create_campaign_draft_with_html`. Returns a previewUrl + approveUrl. Merchant approves in the dashboard or via `send_campaign`.
- **Reusable template**: `create_email_template`. Returns a template id. Future campaigns can be built from it via `get_email_template` → adapt → `create_campaign_draft_with_html`.

Always offer the template path if the merchant says "I'll want to do this every week".

## Forbidden patterns (sanitizer will strip)

- `<script>` — stripped silently
- `<iframe>` — stripped
- Inline JavaScript event handlers (`onclick`, `onmouseover`, etc.) — stripped
- External CSS (`<link rel="stylesheet">`) — stripped
- `<form>` — stripped
- Custom elements / web components — stripped
- Tracking pixels with auth headers — won't load in clients

Use `validate_email_html` (server-side sanitizer dry-run) to confirm an HTML draft before persisting if you're unsure.

## After persistence

Three useful follow-ups to offer the merchant:

1. **Preview**: open the previewUrl in browser
2. **Schedule**: `update_campaign_draft` with a `scheduledAt` (ISO datetime)
3. **Send now with safety net**: `send_campaign` enqueues with a 30-second undo window — abort with `cancel_pending_action`

## Anti-patterns

- Never inline customer data (email, full name) into the *subject line* unless they explicitly asked. The unsubscribe link still lives in the footer; that's enough personalization for most cases.
- Never auto-send. Always return the draft id + preview URL and let the merchant approve.
- Never claim "this won't go to spam" — that's outside our control. Surface the size + deliverability tips and let the merchant decide.
