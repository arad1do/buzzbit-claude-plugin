---
name: buzzbit-team-manager
description: Manage workspace members, roles, and permissions. Loads when the user asks about team / users / invites / roles / permissions / RBAC, or asks to add / remove / change someone's access. Uses list_team_members, invite_team_member, update_member_role, remove_team_member, list_roles_and_permissions.
---

# BuzzBit Team Manager

When the user wants to manage who can do what inside the workspace, load this skill.

## When to load

- "Who's on my team?"
- "Invite alice@agency.com as a viewer"
- "Make Bob an editor"
- "Remove the contractor we hired last month"
- "What permissions does the Editor role have?"
- "Create a custom role for our designer"

## Workflow

### 1. Always start with the role catalog

Before changing anyone's role, fetch the catalog:

```
list_roles_and_permissions
  → returns builtInRoles[] with default permissions per MemberRole
  → returns customRoles[] for this workspace
  → returns allPermissions[] (the full Permission enum, 45+ values)
```

This tells you exactly which permissions each role grants. Don't guess.

### 2. Inviting someone

```
invite_team_member { email, role?, customRoleId? }
```

Defaults to `VIEWER` if `role` is omitted. If the merchant says "give them edit access", check `list_roles_and_permissions.builtInRoles[EDITOR].permissions` first — confirm Editor actually has the perms they expect, not less.

If the email already exists in the workspace (active OR invited), the tool returns `CONFLICT` with the existing member id. Tell the merchant — don't try to "re-invite" by some other path; they need to revoke the existing invite first.

### 3. Changing roles — the OWNER guard

The tool **refuses** to demote the sole OWNER (`LAST_OWNER` error). Workflow when the merchant wants to demote the OWNER:

1. Confirm there's another OWNER. `list_team_members --role=OWNER` (filter implicitly via the result).
2. If only one OWNER exists, propose: "Promote `<some-existing-admin>` to OWNER first, then I can demote you."
3. Only then call `update_member_role`.

### 4. Removing someone

`remove_team_member` goes through the 30-second undo queue. Confirm with the merchant before calling, then surface the `actionId` so they can abort with `cancel_pending_action` if they change their mind.

Same OWNER guard: refuses to remove the sole OWNER. Same workflow as demotion above.

### 5. Custom roles

The 4 built-in roles (OWNER/ADMIN/EDITOR/VIEWER) cover most cases. For granular needs (e.g. "marketer who can only manage flows + campaigns, not billing"), the merchant creates a `CustomRole` in the UI. You can read existing ones via `list_roles_and_permissions.customRoles[]` and reference them by id in `invite_team_member` or `update_member_role`.

## Anti-patterns

- **Never** suggest "just make them OWNER, it's easier" — that bypasses the workspace's permission model.
- **Never** revoke an invitation without telling the merchant — they may have just sent the email.
- When listing team, **do not** include `inviteToken` values in your response. They're returned by `invite_team_member` once at creation; treat them like one-time secrets afterwards.

## Privacy

The `list_team_members` response includes user `email`. When showing it to the merchant, that's fine (they own the workspace). Do not include it in any output destined for a customer-facing channel (campaign drafts, social posts, etc).

## Follow-ups to offer

After a role change:
- "Want me to send them a quick note explaining their new permissions?" — `create_dm_draft` or a manual email
- "Want to audit other Admins / Editors at the same time?" — `list_team_members --status=ACTIVE`

After a removal:
- "Want to revoke any API keys they may have generated?" — `list_api_keys` (currently dashboard-only)
- "Want to log this in your CRM board?" — `create_board_item` on a "Team changes" board
