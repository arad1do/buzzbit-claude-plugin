---
name: buzzbit-content-creator
description: Activates when generating email content, social posts, or popup copy for BuzzBit X. Trigger on requests like "write 10 Instagram posts" or "draft a launch email".
---

# BuzzBit X Content Creator

When the merchant asks for marketing copy, you generate the content; BuzzBit X stores and ships it. This skill keeps your output on-brand and on-format.

## Always start with brand context

```
get_workspace_brand
```

Returns:
- `brandVoice` — one of CASUAL, PROFESSIONAL, LUXURY, URGENT, FRIENDLY, FUNNY
- `primaryColor`, `secondaryColor`, `fontFamily` — for HTML emails
- `companyName`, `companyAddress` — required in compliant marketing emails (Israeli law, GDPR)
- `senderName`, `replyToEmail` — sender display info
- `unsubscribeUrl`, `privacyPolicyUrl` — legal footer links

Match the voice in everything you write.

## Email campaigns

**Block-based draft** (text body, no HTML):
```
create_campaign_draft({
  name: "Black Friday Sneak Peek",
  type: "email",
  subject: "Hey {{first_name}}, our biggest sale starts Friday",
  content: "<plain text or markdown body>",
  segmentId: "<optional>"
})
```

**Raw HTML** (you generate the HTML, sanitizer auto-injects unsubscribe footer):
```
validate_email_html({html: <your html>})  // first — review warnings
create_campaign_draft_with_html({name, subject, html: <your sanitized html>, segmentId})
```

**Bulk** (up to 50 drafts in one call — for campaign series, A/B variants, etc.):
```
bulk_create_email_campaigns({campaigns: [<array of {name, subject, content|html, ...}>]})
```

### Email writing rules

- **Subject line ≤ 50 chars.** Beyond 50 truncates on most clients.
- **Preheader (the line after subject in inbox).** First 90 chars of body. Make it worth reading.
- **First sentence.** Personal hook, not "Welcome to our newsletter".
- **Single CTA** above the fold. Multiple CTAs split attention.
- **Body length 80–250 words** for promotional, 200–400 for newsletter, 400–600 for long-form story.
- **Always personalize** with `{{first_name}}` (and a fallback like "there").
- **Footer** is auto-injected by the sanitizer if missing — but if you write the HTML yourself, include `{{unsubscribe_url}}`.

## Social posts

```
create_social_post_draft({
  content: "<caption>",
  mediaUrls: [<image/video URLs>],
  platforms: ["INSTAGRAM", "FACEBOOK"],
  accountIds: ["<social account ids — fetch via list_social_posts to learn what's connected>"],
  scheduledAt: "<ISO datetime>"
})
```

**Bulk** (up to 30 posts per call — for content calendars):
```
bulk_create_social_posts({posts: [<array>]})
```

### Social writing rules

- **Instagram caption**: hook in first line (it's all that shows in feed), 150–300 char body, hashtags at end (5–10 relevant tags).
- **Facebook**: 50–80 char captions get highest engagement, but 200–400 is fine for value content.
- **TikTok**: 80–120 char caption, 3–5 hashtags, hook in first 3 words.
- **LinkedIn**: 150–300 word professional, no hashtags in body (1–3 at end).
- **Pinterest**: title 100 chars + description 250 chars, both keyword-rich.

The merchant uploads media themselves OR you call `upload_media({filename, mimeType, base64})` if they pasted/generated assets you can attach.

## Popups

Popups have `design`, `content`, `triggers`, `targeting` JSON shapes. The simplest pattern:

```
create_popup_draft({
  name: "Welcome Discount",
  design: {<colors, layout — match brand>},
  content: {headline: "Get 10% off your first order", subhead: "...", cta: "Claim 10% off"},
  triggers: {exitIntent: true, scrollPercent: 50},
  targeting: {urlMatch: ["/", "/products/*"], device: ["desktop","mobile"]}
})
```

Don't over-target — broad triggers + simple content beats clever logic.

## Anti-patterns

- **Don't promise discounts without `create_discount_code` first.** Code "WELCOME10" in copy without the code existing in the system = broken UX.
- **Don't generate >500 words for promotional emails.** People skim.
- **Don't use false urgency** ("Last 3 hours!") unless it's actually true. The brand voice may be URGENT but the facts must be honest.
- **Don't skip `validate_email_html`** for HTML campaigns. Even your "clean" output may include disallowed tags or missing unsubscribe link.
