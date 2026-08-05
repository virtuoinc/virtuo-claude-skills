---
name: meeting-recap
description: >
  Retrieves and summarizes meeting transcripts from Microsoft Teams or Granola,
  extracting key decisions, action items, and next steps. Creates Asana tasks
  for each action item, sends a formatted recap email to attendees, and
  optionally posts a summary to a Teams channel. Invoke with /meeting-recap.
---

# Summarizing Meeting Transcripts

## MCP Requirements

| MCP Server      | Tool                                        | Purpose                                          |
|-----------------|---------------------------------------------|--------------------------------------------------|
| `Microsoft 365` | `Microsoft 365:get_online_meetings`         | Find the specified meeting                       |
| `Microsoft 365` | `Microsoft 365:get_meeting_transcript`      | Retrieve Teams meeting transcript                |
| `Microsoft 365` | `Microsoft 365:get_calendar_events`         | Locate meeting via calendar if needed            |
| `Microsoft 365` | `Microsoft 365:send_mail`                   | Send recap email to attendees                    |
| `Microsoft 365` | `Microsoft 365:send_channel_message`        | Post recap to a Teams channel (optional)         |
| `Granola`       | `Granola:search_transcripts`                | Find transcript in Granola by meeting name/date  |
| `Granola`       | `Granola:get_transcript`                    | Retrieve full Granola transcript                 |
| `Asana`         | `Asana:create_task`                         | Create one task per action item                  |

> Tool names are based on common MCP conventions — verify exact names against your configured MCP servers in Claude Desktop.

## When to use this skill

Use this skill when:
- The user asks to summarize, recap, or get action items from a past meeting
- The user asks to send a meeting recap to attendees
- The user references a meeting by name, date, or attendees and wants a summary

Do NOT use this skill when:
- The user wants to prepare before a meeting (use `/meeting-prep`)
- The user asks for a general email or calendar summary (use `/daily-briefing`)
- The user wants to capture tasks from freeform text (email, Slack, dictated list) without a full meeting recap (use `/asana-task-capture`)

## Instructions

### Step 1: Find the meeting

If the user named a specific meeting, date, or person:
1. Call `Microsoft 365:get_online_meetings` searching by subject or date range
2. If not found in Teams, call `Granola:search_transcripts` with the meeting name, date, or attendee names
3. If still not identified, call `Microsoft 365:get_calendar_events` to find recent past events matching the description

If no meeting is specified, fetch the most recently completed calendar event with online meeting details.

Confirm with the user if multiple matches are found.

### Step 2: Retrieve the transcript

Try both sources in parallel:
- `Microsoft 365:get_meeting_transcript` using the meeting ID from Step 1
- `Granola:search_transcripts` then `Granola:get_transcript` using the meeting name/date

If both return content, prefer the more complete transcript. Note the source.

If no transcript is available, summarize from the meeting recording metadata or inform the user that no transcript was found.

### Step 3: Extract structured content

Analyze the full transcript and extract:

1. **Summary** (3–5 sentences): what the meeting covered and the overall outcome
2. **Key Decisions**: bulleted list of concrete decisions made
3. **Action Items**: list each as:
   - What needs to be done
   - Who is responsible (name from transcript)
   - Due date if mentioned; otherwise leave blank
3. **Open Questions**: unresolved items that need follow-up
4. **Next Meeting**: date/time if scheduled during the meeting

### Step 4: Create Asana tasks for action items

For each action item extracted in Step 3, call `Asana:create_task` with:
- `name`: the action item description
- `assignee`: the responsible person's name or email (if identifiable)
- `due_on`: due date if mentioned (ISO format: YYYY-MM-DD), omit if not stated
- `notes`: source context — meeting name, date, relevant quote from transcript

Confirm each task was created. If an assignee cannot be matched to an Asana user, create the task unassigned and note it in the recap.

### Step 5: Present recap to user and offer distribution

Show the full structured recap to the user.

Then ask:
- **"Send recap email to attendees?"** — if yes, compose and send via `Microsoft 365:send_mail`:
  - To: all meeting attendees (from the calendar event)
  - Subject: "Recap: [Meeting Subject] — [Date]"
  - Body: the structured recap formatted in plain text
- **"Post to a Teams channel?"** — if yes, ask which channel, then call `Microsoft 365:send_channel_message`

## Keywords

meeting recap, summarize meeting, action items, meeting summary, what happened in, meeting notes, send recap, recap email, transcript summary, decisions from meeting
