---
name: meeting-prep
description: >
  Prepares a comprehensive pre-meeting briefing by pulling together attendee
  context, past communications, meeting transcripts, CRM history, and relevant
  documents from Microsoft 365, HubSpot, and Granola. Produces a structured
  briefing card covering who's attending, relationship history, recent email
  threads, past meeting notes, CRM deal context for external contacts, and
  relevant files. Invoke with /meeting-prep.
---

# Preparing for Meetings

## MCP Requirements

| MCP Server      | Tool                                        | Purpose                                           |
|-----------------|---------------------------------------------|---------------------------------------------------|
| `Microsoft 365` | `Microsoft 365:get_calendar_events`         | Fetch upcoming or specified meeting               |
| `Microsoft 365` | `Microsoft 365:get_user`                    | Look up internal attendee profile and org info    |
| `Microsoft 365` | `Microsoft 365:search_messages`             | Find recent sent and received emails with attendee|
| `Microsoft 365` | `Microsoft 365:get_online_meetings`         | Find past meetings with attendee                  |
| `Microsoft 365` | `Microsoft 365:get_meeting_transcript`      | Retrieve Teams meeting transcript                 |
| `Microsoft 365` | `Microsoft 365:search_drive_files`          | Search OneDrive and SharePoint for relevant files |
| `Hubspot`       | `Hubspot:search_contacts`                   | Look up external attendee in CRM                  |
| `Hubspot`       | `Hubspot:get_contact_activities`            | Get recent CRM activity for external contact      |
| `Granola`       | `Granola:search_transcripts`                | Search Granola for past meeting notes/transcripts |

> Tool names are based on common MCP conventions — verify exact names against your configured MCP servers in Claude Desktop.

## When to use this skill

Use this skill when:
- The user asks to "prep for", "prepare for", or "brief me on" a meeting
- The user asks "who am I meeting with today/tomorrow/at [time]"
- The user asks for context on a specific person before a call
- The user asks "what should I know before my meeting with [name]"

Do NOT use this skill when:
- The user wants to summarize a meeting that already happened (use `/meeting-recap`)
- The user asks about their general calendar without preparation context

## Instructions

### Step 1: Identify the meeting

If the user named a specific meeting or person, use that. Otherwise, call `Microsoft 365:get_calendar_events` to fetch the next upcoming meeting or today's meetings, filtering for meetings with external participants or the next unstarted event.

Ask the user to confirm which meeting to prepare for if multiple are returned.

Extract from the event:
- Meeting subject and scheduled time
- Full list of attendee email addresses
- Meeting body/agenda (if any)

### Step 2: Classify each attendee

For each attendee email address, determine if they are **internal** or **external**:
- **Internal**: email domain matches the organizer's domain
- **External**: any other domain

Skip the user themselves in all lookups.

### Step 3: Look up attendee context (run all lookups in parallel)

**For internal attendees** — call in parallel:
- `Microsoft 365:get_user` with the attendee's email to get name, title, department, manager, and office location

**For external attendees** — call in parallel:
- `Hubspot:search_contacts` using the attendee's email address to find their CRM record
- If found, call `Hubspot:get_contact_activities` to get recent notes, calls, emails, and deal associations

**For all attendees** — call in parallel:
- `Microsoft 365:search_messages` filtering by the attendee's email address (both sent and received, last 90 days, limit 10) to surface recent email threads
- `Microsoft 365:get_online_meetings` to find past Teams meetings that included this attendee
  - For each past meeting found, call `Microsoft 365:get_meeting_transcript` to retrieve the transcript
- `Granola:search_transcripts` using the attendee's name and/or company to find Granola meeting notes

**Also in parallel** — search for relevant documents:
- `Microsoft 365:search_drive_files` using the meeting subject, attendee names, and attendee company names as search terms (limit 5 results)

### Step 4: Synthesize the briefing card

Compile everything into a structured briefing card. Use this format:

---

## Meeting Brief: [Subject] — [Date & Time]

### Attendees

For each attendee:

**[Full Name]** — [Title, Company]
- *Internal/External*
- **Background**: [role, department, manager chain for internal; company, deal stage for external]
- **CRM Context** *(external only)*: [last touchpoint, open deals, recent activity from HubSpot]
- **Recent Emails**: [1–3 sentence summary of recent email threads]
- **Past Meetings**: [bullet list of past meeting dates and key topics from transcripts/Granola notes]

### Agenda
[Meeting body/agenda if provided, or "No agenda found"]

### Relevant Files
[Linked list of relevant documents found, with brief description of each]

### Suggested Talking Points
Based on all context gathered, suggest 2–3 specific, actionable conversation starters or items to address.

---

If no data is found for a section, omit that section rather than showing "none found".

### Step 5: Offer follow-up actions

After presenting the brief, offer:
- Save the brief as a file in OneDrive (`Microsoft 365:upload_file`)
- Add notes to the calendar event (`Microsoft 365:update_calendar_event`)

## Keywords

meeting prep, prepare for meeting, brief me, who am I meeting, meeting briefing, pre-meeting, next meeting, meeting context, meeting with, call prep, before my meeting
