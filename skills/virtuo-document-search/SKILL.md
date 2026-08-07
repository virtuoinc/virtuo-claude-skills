---
name: virtuo-document-search
description: >
  Searches personal OneDrive and Virtuo SharePoint for documents, files, and
  site content matching a topic, project, or keyword, then summarizes the most
  relevant results with direct links. Returns ranked results with AI-generated
  summaries so the user can identify the right file without opening each one.
  Invoke with /virtuo-document-search.
---

# Researching Documents

## MCP Requirements

| MCP Server      | Tool                                    | Purpose                                         |
|-----------------|-----------------------------------------|-------------------------------------------------|
| `Microsoft 365` | `Microsoft 365:search_drive_files`      | Search OneDrive files by keyword/topic          |
| `Microsoft 365` | `Microsoft 365:search_sites`            | Search SharePoint sites and pages               |
| `Microsoft 365` | `Microsoft 365:get_drive_file_content`  | Read file content for summarization             |
| `Microsoft 365` | `Microsoft 365:upload_file`             | Save research summary to OneDrive (optional)    |


## When to use this skill

Use this skill when:
- The user asks to find, search for, or look up files, documents, or SharePoint content
- The user wants to know what documents exist on a given topic or project
- The user needs a summary of file contents without opening the files manually

Do NOT use this skill when:
- The user asks about email content (use `/email-triage`)
- Document search is part of meeting prep (use `/meeting-prep`, which handles this internally)

## Instructions

### Step 1: Understand the search intent

Extract from the user's request:
- **Search terms**: topic, project name, document name, person's name, or keyword
- **Scope hint**: specific SharePoint site, team, or folder if mentioned; otherwise search broadly
- **File type hint**: if the user specifies (e.g. "presentations", "spreadsheets", "proposals")

If the search intent is ambiguous, ask one clarifying question before proceeding.

### Step 2: Search in parallel

Call both sources simultaneously:

**OneDrive / personal files** — call `Microsoft 365:search_drive_files` with:
- `query`: the extracted search terms
- `top`: 10
- Include `file.mimeType`, `name`, `webUrl`, `lastModifiedDateTime`, `lastModifiedBy`, `size`

**SharePoint** — call `Microsoft 365:search_sites` with:
- `query`: the same search terms
- `top`: 10
- Include site name, URL, last modified date

If the user specified a file type, add a type filter (e.g. `application/vnd.openxmlformats-officedocument.presentationml.presentation` for PowerPoint).

### Step 3: Rank and summarize results

Score results by relevance using:
1. Keyword match density in title vs body
2. Recency (more recent = higher)
3. Author relevance (files by or shared with the user rank higher)

For the top 5 results:
- If the file is a document type (Word, PDF, PowerPoint), call `Microsoft 365:get_drive_file_content` to retrieve enough content to write a meaningful 2–3 sentence summary
- For other file types, use the metadata and any available preview text

### Step 4: Present results

Format the output as:

---

## Document Research: "[search query]"

Found [N] results across OneDrive and SharePoint.

### Most Relevant

**1. [File Name]** — [File type]
- 📁 [Location / SharePoint site or OneDrive]
- 🕒 Last modified [date] by [name]
- 🔗 [Open file](webUrl)
- **Summary**: [2–3 sentences describing what the file contains and why it's relevant]

**2. [File Name]** ...

*(Repeat for top 5)*

### Also Found
[List remaining results as single-line entries: name, location, date, link — no summaries]

---

### Step 5: Offer to save research summary

Ask: "Would you like me to save this research summary as a document in OneDrive?"

If yes, call `Microsoft 365:upload_file` to create a Markdown or text file in the user's OneDrive with the full results.

## Keywords

find documents, search SharePoint, look up files, find files about, what files exist, document search, OneDrive search, search for files, find the document, files related to
