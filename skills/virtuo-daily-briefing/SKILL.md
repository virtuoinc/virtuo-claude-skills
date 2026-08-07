---
name: virtuo-daily-briefing
description: >
  Delivers a prioritized daily briefing combining today's calendar, unread
  emails since the last workday, and unread Teams chat and channel messages.
  Synthesizes everything into a single digest to start or end the workday with
  full situational awareness. Covers calendar events, email highlights, and
  Teams activity in one response. Invoke with /virtuo-daily-briefing.
---

# Briefing the Daily Workday

## MCP Requirements

| MCP Server      | Tool                                    | Purpose                                         |
|-----------------|-----------------------------------------|-------------------------------------------------|
| `Microsoft 365` | `Microsoft 365:get_calendar_events`     | Fetch today's (or tomorrow's) scheduled events  |
| `Microsoft 365` | `Microsoft 365:list_messages`           | Fetch unread emails since last workday          |
| `Microsoft 365` | `Microsoft 365:get_message`             | Get full body of high-priority emails           |
| `Microsoft 365` | `Microsoft 365:list_chats`              | Fetch recent Teams chat conversations           |
| `Microsoft 365` | `Microsoft 365:get_chat_messages`       | Get messages from a specific Teams chat         |
| `Microsoft 365` | `Microsoft 365:list_channel_messages`   | Fetch unread Teams channel messages             |


## When to use this skill

Use this skill when:
- The user asks for a morning briefing, daily digest, or "what's my day look like"
- The user asks to be caught up after time away (weekend, holiday, travel)
- The user wants a combined view of calendar + email + Teams in one response

Do NOT use this skill when:
- The user only wants email (use `/email-triage`)
- The user wants to prepare for a specific meeting (use `/meeting-prep`)
- The user wants a summary of a past meeting (use `/meeting-recap`)

## Instructions

### Step 1: Determine the time window

- **Morning briefing / start of day**: window = end of last workday until now
- **End of day summary**: window = start of today until now
- **Catch-up after time away**: use the number of days missed (ask if unclear)
- **Tomorrow preview**: calendar only, no email/Teams lookups needed

Default to "since end of yesterday" if unspecified.

### Step 2: Fetch all data in parallel

Run all three fetches simultaneously:

**Calendar** — call `Microsoft 365:get_calendar_events` with:
- `startDateTime`: start of today (or tomorrow for a preview)
- `endDateTime`: end of today (or tomorrow)
- `orderby`: start.dateTime asc
- `top`: 25

**Email** — call `Microsoft 365:list_messages` with:
- `filter`: `isRead eq false and receivedDateTime ge [window start]`
- `orderby`: receivedDateTime desc
- `top`: 50
- `select`: sender, subject, receivedDateTime, importance, bodyPreview

**Teams** — call `Microsoft 365:list_chats` to get active chat IDs, then call `Microsoft 365:get_chat_messages` for each active chat (last 24–48 hours). Also call `Microsoft 365:list_channel_messages` for key channels the user participates in.

For any high-importance email, call `Microsoft 365:get_message` to get the full body before summarizing.

### Step 3: Synthesize into a prioritized briefing

Format the output as:

---

## Daily Briefing — [Day, Date]

### 📅 Your Day ([N] events)
List each calendar event in order:
- **[Time]** — [Meeting Subject] with [Attendee names or count]
  *(flag if back-to-back, if preparation might be needed, or if it has no agenda)*

### 📬 Email Highlights ([N] unread)
**Needs your attention:**
- **[Sender]** — *[Subject]*: [1–2 sentence summary of what's needed]

**FYI / should read:**
- [Subject] from [Sender] — [1 sentence]

*[N] low-priority emails (notifications, newsletters) not shown*

### 💬 Teams Activity
**Direct messages:**
- **[Person]**: [summary of what they said or asked]

**Channel highlights:**
- **[Channel name]**: [summary of key thread or discussion]

### ⚡ Top Priorities
Based on the above, call out 2–3 specific things that need attention first — ordered by urgency.

---

Keep each section concise. If a section has nothing to report, omit it entirely rather than stating "nothing to show."

### Step 4: Offer follow-up

After the briefing, offer:
- "Prep for your first meeting?" — run `/meeting-prep` for the first calendar event
- "Triage your inbox in detail?" — run `/email-triage`

## Keywords

morning briefing, daily digest, start my day, what's my day, catch me up, end of day, daily summary, today's schedule, what did I miss, workday briefing
