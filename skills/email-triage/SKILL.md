---
name: email-triage
description: >
  Fetches recent or unread emails from Microsoft 365, groups them by priority,
  and delivers a tiered summary so the user can quickly identify what needs
  action. Optionally flags, moves, marks as read, or replies to emails.
  Handles both personal and shared mailboxes. Invoke with /email-triage.
---

# Triaging Emails

## MCP Requirements

| MCP Server      | Tool                                   | Purpose                                        |
|-----------------|----------------------------------------|------------------------------------------------|
| `Microsoft 365` | `Microsoft 365:list_messages`          | Fetch emails from inbox with filters           |
| `Microsoft 365` | `Microsoft 365:get_message`            | Retrieve full email body for context           |
| `Microsoft 365` | `Microsoft 365:send_mail`              | Send a reply on behalf of the user             |
| `Microsoft 365` | `Microsoft 365:update_message`         | Flag, mark as read, or move an email           |
| `Microsoft 365` | `Microsoft 365:list_shared_messages`   | Access shared/delegated mailboxes (optional)   |

> Tool names are based on common MCP conventions — verify exact names against your configured MCP servers in Claude Desktop.

## When to use this skill

Use this skill when:
- The user asks to triage, summarize, or catch up on email
- The user asks what's urgent or needs attention in their inbox
- The user wants to action specific emails (reply, flag, move)

Do NOT use this skill when:
- The user asks for a full daily digest including calendar and Teams (use `briefing-daily-workday`)
- The user wants to find emails about a specific person as part of meeting prep (use `preparing-for-meetings`)

## Instructions

### Step 1: Fetch emails

Call `Microsoft 365:list_messages` with:
- `folder`: inbox
- `filter`: `isRead eq false` (unread only by default)
- `orderby`: `receivedDateTime desc`
- `top`: 50
- `select`: sender, subject, receivedDateTime, importance, hasAttachments, bodyPreview

If the user specified a time range (e.g. "since yesterday", "last 2 days"), add a `receivedDateTime ge [ISO date]` filter.

If the user mentioned a shared mailbox, also call `Microsoft 365:list_shared_messages` for that mailbox.

### Step 2: Categorize into priority tiers

Analyze subject lines, senders, importance flags, and body previews. Group emails into three tiers:

**Tier 1 — Needs action (respond or decide)**
- Flagged as high importance
- Direct questions addressed to the user
- Time-sensitive requests (deadlines, approvals, confirmations)
- Emails from direct reports, senior leadership, or key clients

**Tier 2 — Should read (informational, no immediate action)**
- FYI threads, project updates, status reports
- Meeting invites not yet accepted

**Tier 3 — Low priority (newsletters, notifications, automated)**
- Bulk mail, system notifications, newsletters
- Threads where the user is only CC'd with no clear action

For any Tier 1 email where the full body is needed to confirm urgency, call `Microsoft 365:get_message` with that message's ID.

### Step 3: Present tiered summary

Format the output as:

---

## Email Triage — [date/time range]

**[N] emails reviewed** | [N] unread

### 🔴 Needs Action ([N])
- **[Sender]** — *[Subject]* ([time ago])
  [1–2 sentence summary of what's needed and any deadline]

### 🟡 Should Read ([N])
- **[Sender]** — *[Subject]* ([time ago])
  [1 sentence summary]

### ⚪ Low Priority ([N])
[grouped count by type, e.g. "8 newsletters, 4 system notifications"]

---

### Step 4: Offer actions

After presenting the triage, offer the following for each Tier 1 email:
- **Reply** — draft and send a reply via `Microsoft 365:send_mail`
- **Flag** — call `Microsoft 365:update_message` to set `flag.flagStatus: flagged`
- **Mark as read** — call `Microsoft 365:update_message` to set `isRead: true`

For bulk Tier 3 items, offer to mark all as read in one step.

## Keywords

triage inbox, summarize emails, catch up on email, unread emails, urgent emails, what needs attention, email summary, check email, inbox summary, email triage
