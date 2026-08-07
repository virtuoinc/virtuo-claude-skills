---
name: virtuo-meeting-recap
description: >
  Retrieves and summarizes meeting transcripts from Microsoft Teams or Granola,
  extracting key decisions, action items, and next steps. Sends a formatted
  recap email to attendees and optionally posts a summary to a Teams channel.
  Invoke with /virtuo-meeting-recap.
---

# Summarizing Meeting Transcripts

## MCP Requirements

| MCP Server      | Tool                               | Purpose                                         |
|-----------------|------------------------------------|-------------------------------------------------|
| `Microsoft 365` | `OnlineMeetings.Read`              | Find the specified meeting                      |
| `Microsoft 365` | `OnlineMeetingTranscript.Read.All` | Retrieve Teams meeting transcript               |
| `Microsoft 365` | `Calendars.Read`                   | Locate meeting via calendar if needed           |
| `Microsoft 365` | `Mail.ReadWrite`                   | Send recap email to attendees                   |
| `Microsoft 365` | `ChannelMessage.Send`              | Post recap to a Teams channel (optional)        |
| `granola`       | `granola-query_granola_meetings`   | Search for meeting in Granola by name/date      |
| `granola`       | `granola-get_meeting_transcript`   | Retrieve full Granola meeting transcript        |


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
1. Call `OnlineMeetings.Read` searching by subject or date range
2. If not found in Teams, call `granola-query_granola_meetings` with the meeting name, date, or attendee names
3. If still not identified, call `Calendars.Read` to find recent past events matching the description

If no meeting is specified, fetch the most recently completed calendar event with online meeting details.

Confirm with the user if multiple matches are found.

### Step 2: Retrieve the transcript

Try both sources in parallel:
- `OnlineMeetingTranscript.Read.All` using the meeting ID from Step 1
- `granola-query_granola_meetings` then `granola-get_meeting_transcript` using the meeting name/date

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

### Step 4: Present recap to user and offer distribution

Show the full structured recap to the user.

Then ask:
- **"Send recap email to attendees?"** — if yes, compose and send via `Mail.ReadWrite`:
  - To: all meeting attendees (from the calendar event)
  - Subject: "Recap: [Meeting Subject] — [Date]"
  - Body: the structured recap formatted in plain text
- **"Post to a Teams channel?"** — if yes, ask which channel, then call `ChannelMessage.Send`

## Keywords

meeting recap, summarize meeting, action items, meeting summary, what happened in, meeting notes, send recap, recap email, transcript summary, decisions from meeting
