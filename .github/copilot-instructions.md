# Virtuo Claude Skills — Agent Instructions

This repo is a Claude Desktop **plugin marketplace**. The Git URL is registered
in the Claude Desktop app (Settings → Plugins & skills → Plugin Marketplaces),
and the app re-clones it periodically to deliver skill updates to all users.

Each skill in `skills/` is an Agent Skill: a folder containing a `SKILL.md`
file. Skills are primarily invoked via slash command (`/skill-name`) in Claude
Desktop. The skill `name` is the slash command — keep it short and natural.
Skills explicitly state which MCP servers and tools are required — Claude uses
those tools to complete the task.

## Repo structure

```
virtuo-claude-skills/
├── .claude-plugin/
│   └── marketplace.json        ← plugin catalog; update when adding a skill
├── skills/
│   ├── _template/
│   │   └── SKILL.md            ← copy this to create a new skill
│   └── <skill-name>/
│       ├── SKILL.md            ← required
│       └── REFERENCE.md        ← optional; link from SKILL.md, 1 level deep
└── .github/
    └── copilot-instructions.md ← this file
```

## Adding a new skill

1. Copy `skills/_template/` to `skills/<skill-name>/` using a short noun-phrase
   or noun+verb name that reads naturally as a slash command
   (e.g. `meeting-prep`, `email-triage`, `document-search`).
2. Fill in `SKILL.md` following the authoring rules below.
3. Add a plugin entry to `.claude-plugin/marketplace.json` (see format below).

### Adding a marketplace entry

When a new skill folder is ready, append an entry to the `plugins` array in
`.claude-plugin/marketplace.json`:

```json
{
  "name": "<skill-name>",
  "description": "<one-sentence description for the marketplace UI>",
  "source": "./",
  "strict": false,
  "skills": [
    "./skills/<skill-name>"
  ]
}
```

`source: "./"` and `strict: false` means the marketplace defines everything;
no separate `plugin.json` is needed inside the skill folder.

When grouping related skills into a single installable plugin (e.g. all HR
skills), list multiple skill paths under one entry's `skills` array instead of
creating separate entries.

## SKILL.md authoring rules

### YAML frontmatter (required)

```yaml
---
name: your-skill-name       # lowercase, hyphens only, ≤ 64 chars
description: "..."          # ≤ 1024 chars, no XML tags, third person
---
```

- `name`: lowercase letters, digits, and hyphens only. No spaces. No reserved
  words (`anthropic`, `claude`).
- `description`: **must state WHAT the skill does.** Written in third person
  ("Processes...", "Generates...", not "I can..." or "You can..."). End with
  `Invoke with /skill-name.` Do NOT fill the description with natural-language
  trigger phrases like "Use when asked for X" — skills are slash-command-first.
  This text is shown in the Claude Desktop directory and loaded into the system
  prompt; keep it focused on capability, not invocation conditions.

### MCP Requirements section (mandatory)

Every SKILL.md must include an `## MCP Requirements` table listing:
- The MCP server name exactly as configured in Claude Desktop
- Each tool in `ServerName:tool_name` format
- A one-line description of what each tool is used for

Claude must use the exact tool references from this table in its instructions.
Do not tell Claude to "use Jira" — tell it to call `Jira:create_issue`.

### Body content rules

- Keep the total SKILL.md body under **500 lines**.
- Move detailed reference material to a separate `REFERENCE.md` and link to it
  from SKILL.md: `[details](REFERENCE.md)`. Keep references **one level deep**
  — do not chain references across multiple files.
- Use forward slashes in all file paths (never backslashes).
- Use consistent terminology throughout — pick one term and never vary it.
- Do not include time-sensitive information (dates, "as of X version", etc.).
- Write in third person. Never "I will..." or "You should...".

### Skill naming (slash-command-first)

Use short noun-phrase or action-noun names that read naturally after `/`:
`meeting-prep`, `email-triage`, `document-search`, `daily-briefing`.

Avoid: `jira`, `meetings`, `hr-tools` (too vague), `claude-jira-helper`
(reserved word), `CreateJiraTicket` (wrong case), `preparing-for-meetings`
(gerund-form — awkward as a slash command).

### MCP tool reference format

Always use fully qualified tool names: `ServerName:tool_name`.

Examples:
- `Jira:create_issue` — not just `create_issue`
- `GitHub:create_pull_request` — not `create PR`
- `Slack:post_message` — not `send a Slack message`

Without the server prefix, Claude may pick the wrong tool when multiple MCP
servers are active.

### Degrees of freedom

Match instruction specificity to the fragility of the task:

- **Exact sequence required** (e.g. a multi-step API flow): list numbered steps
  with explicit tool calls and required parameters.
- **Context-dependent** (e.g. tone of a message): give principles and let
  Claude decide.
- **Multiple valid paths**: provide a default and note the alternative.

### Checklist before committing

- [ ] `name` is lowercase/hyphens, ≤ 64 chars, no reserved words
- [ ] `description` states what the skill does, is third person, ends with `Invoke with /name`, ≤ 1024 chars
- [ ] `description` does NOT contain natural-language trigger phrases ("Use when asked for...")
- [ ] `## MCP Requirements` table is present and uses `Server:tool` format
- [ ] Body is under 500 lines
- [ ] All file references are one level deep
- [ ] No Windows-style backslash paths
- [ ] Skill folder is listed in `.claude-plugin/marketplace.json`

## Anti-patterns to avoid

| Anti-pattern | Fix |
|---|---|
| `description: "Helps with Jira"` | Be specific: what actions, which objects |
| `description` full of "Use when asked for X" phrases | Remove them — skills are slash-command-first; end description with `Invoke with /name` |
| `name: preparing-for-meetings` | Use slash-command-friendly name: `meeting-prep` |
| "Use the Jira tool to create an issue" | "Call `Jira:create_issue` with `summary: ...`" |
| Nested `SKILL.md → guide.md → details.md` | Flatten to `SKILL.md → details.md` only |
| 800-line SKILL.md | Split into `SKILL.md` + `REFERENCE.md` |
| `scripts\helper.py` | `scripts/helper.py` |
| Vague skill name: `tools` | Specific: `document-search`, `email-triage` |
